---
title: "Playing with /proc"
date: 2024-12-01 10:00:03 +0530
categories: [linux, beginner]
tags: [linux, Algorithms]
image: https://cdn.jsdelivr.net/gh/rseragon/blog-assets@main/proc/imgs/tuxus_holding_proc.png
---
> Credits for image: [deepai.org](https://deepai.org)

`/proc` is a virtual file system that contains runtime system information like system/process memory, mounted devices, hardware information etc. The kernel saves the state to to a specific set of files which can be utilized by the kernel itself or other programs interacting with the system. These files contain information such as the range of memory, state, file descriptors used by a process, the current CPU's make, model, type, the type of filesystems supported by the kernel etc.

There is a plethora of information that can be derived from the `/proc/` fs. In this article, we'll be focusing on few major components like processes and system state information. The rest of 'em, I'll leave them to your homework.

> Sidenote: I'm writing this article because my project [Kunai](https://github.com/rseragon/Kunai) extensively uses the procfs to manipulate values of running processes. Thought I would share my knowledge to the world as well :)

# Getting system information

Just for the sakes of it, try listing the files present in `/proc`. What do you find? Some weird gibberish like this?

```console
~ ls -F /proc
1/       122692/  204/     249/     433145/  483834/  488875/  779/            interrupts
100/     122717/  21/      25/      434833/  484146/  488885/  78/             iomem
101/     122769/  210/     250/     435738/  484676/  489282/  787/            ioports
102/     124/     211/     251/     437394/  484840/  489309/  79/             irq/
104/     1240/    212/     252/     437396/  484907/  489310/  796/            kallsyms
105/     1242/    213/     253/     439/     484908/  489338/  797/            kcore
106/     1246/    214/     254/     439388/  484913/  489383/  798/            keys
106048/  125/     215/     255/     440172/  485263/  49/      799/            key-users
106059/  1251/    216/     256/     440388/  485500/  5/       800/            kmsg
106060/  1255/    217/     28/      440954/  485965/  52/      801/            kpagecgroup
106062/  1257/    218/     29/      441521/  485967/  53/      803/            kpagecount
106064/  1259/    219/     3/       441929/  486259/  530/     82/             kpageflags
106070/  126/     22/      30/      442536/  486321/  54/      83/             latency_stats
106071/  127/     220/     300/     442967/  486439/  542/     84/             loadavg
106072/  128/     221/     301/     444234/  486515/  546/     844/            locks
106075/  1287/    222/     31/      445444/  486796/  547/     847/            meminfo
106086/  13/      223/     328693/  451067/  486801/  548/     85/             misc
106116/  130/     224/     328797/  452888/  486805/  549/     88/             modules
106120/  1306/    225/     338/     453771/  486870/  55/      89/             mounts@
106130/  1310/    226/     34/      46/      486871/  550/     90/             mtrr
1068/    1317/    227/     344/     461173/  486890/  551/     92/             net@
107/     132/     228/     345/     461205/  486891/  552/     93/             pagetypeinfo
1070/    1338/    229/     35/      463094/  487050/  553/     95/             partitions
1071/    1349/    23/      36/      463134/  487079/  554/     96/             pressure/
1078/    1369/    230/     37/      463240/  487200/  555/     97/             schedstat
1079/    1382/    231/     374123/  467135/  487459/  556/     99/             scsi/
108/     139657/  232/     374147/  467314/  487565/  557/     acpi/           self@
1080/    14/      233/     381/     467349/  487569/  558/     asound/         slabinfo
1081/    140/     234/     382/     467951/  487681/  559/     bootconfig      softirqs
1083/    1406/    235/     4/       467961/  487859/  58/      buddyinfo       stat
1093/    141/     236/     40/      468046/  487860/  59/      bus/            swaps
11/      1411/    237/     402072/  468064/  488038/  6/       cgroups         sys/
110/     143/     238/     41/      468105/  488253/  60/      cmdline         sysrq-trigger
1100/    145297/  239/     410348/  468185/  488255/  61/      config.gz       sysvipc/
111/     149760/  24/      414236/  468786/  488322/  64/      consoles        thread-self@
112/     15/      240/     414278/  469102/  488468/  65/      cpuinfo         timer_list
113/     16/      241/     42/      469240/  488469/  66/      crypto          tty/
1172/    17/      242/     425388/  469307/  488480/  67/      devices         uptime
1200/    18/      243/     425390/  47/      488500/  70/      diskstats       version
122/     19/      244/     425478/  470449/  488572/  7032/    dma             vmallocinfo
122638/  2/       245/     425523/  471494/  488587/  71/      driver/         vmstat
122640/  20/      246/     43/      472196/  488588/  72/      dynamic_debug/  zoneinfo
122651/  2007/    247/     430896/  479324/  488610/  73/      execdomains
122652/  201/     248/     431463/  48/      488677/  76/      fb
122654/  202/     248562/  432470/  481539/  488751/  77/      filesystems
122688/  203/     248583/  432490/  483654/  488823/  778/     fs/
```
*Seems to have a lot of folders! But, what are these weird folders with numbers?*

If that's your first question, you are on the right track. All of the folders with numbers on them represents the [process information](#process-info) related to that process id (PID) stored by the kernel.

There are few other interesting files such as [cpuinfo](#proccpuinfo), [meminfo](#procmeminfo), [modules](#procmodules), [mounts](#procmounts), [uptime](#procuptime), [version](#procversion), [swaps](#procswaps). There are also few folders that I find interesting, such as [self/](#procself), [sys/](#procsys).
The above links are all the stuff that I'm going to cover in this article. Feel free to go to the [References](#references) to learn more about procfs :)

## Process info

Each and every folder with a number represents a process in your system. Specifically, this is the `PID`(Process id) of the process. Every process has an ID associated with it, and all of the process state required by the kernel is stored in `/proc/$PID/` folder.

If you check your system processes
```console
~ pstree -p
systemd(1)─┬─NetworkManager(803)─┬─{NetworkManager}(827)
           │                     ├─{NetworkManager}(829)
           │                     └─{NetworkManager}(830)
           ├─Xwayland(106086)─┬─{Xwayland}(106095)
           │                  ├─{Xwayland}(106096)
           │                  ├─{Xwayland}(106097)
           │                  ├─{Xwayland}(106098)
           │                  ├─{Xwayland}(106099)
           │                  ├─{Xwayland}(106100)
           │                  ├─{Xwayland}(106101)
           │                  ├─{Xwayland}(132948)
           │                  ├─{Xwayland}(132949)
           │                  └─{Xwayland}(132950)
           ├─alacritty(414236)─┬─zsh(414278)───nvim(425388)─┬─nvim(425390)─┬─zsh(425478)───bundle(425523)─┬─{bundle}(425559)
           │                   │                            │              │                              ├─{bundle}(425560)
           │                   │                            │              │                              ├─{bundle}(425561)
           │                   │                            │              │                              └─{bundle}(486973)
           │                   │                            │              ├─{nvim}(425392)
           │                   │                            │              ├─{nvim}(487774)
           │                   │                            │              ├─{nvim}(487775)
           │                   │                            │              ├─{nvim}(487776)
           │                   │                            │              ├─{nvim}(487777)
           │                   │                            │              ├─{nvim}(487784)
           │                   │                            │              ├─{nvim}(488323)
           │                   │                            │              └─{nvim}(492949)
           │                   │                            ├─{nvim}(425389)
           │                   │                            └─{nvim}(425391)
           │                   ├─{alacritty}(414238)
           │                   ├─{alacritty}(414239)
           │                   ├─{alacritty}(414240)
           │                   ├─{alacritty}(414241)
           │                   ├─{alacritty}(414265)
           │                   ├─{alacritty}(414266)
           │                   ├─{alacritty}(414267)
           │                   ├─{alacritty}(414268)
```

I've used `pstree` for pretty presentation, you can use a normal `ps` or `top`. But the most important thing that you'll have to notice here is the `PID` of the process.

For example, if you take `systemd(1)`, it's `PID` is 1. It is the init process which initializes other processes and daemons in the system.
Let's have a look at the process 1

### /proc/1
Listing the files in `/proc/1` gives us a output similar to this
```bash
/proc/1
❯ ls
 arch_status   clear_refs           cpuset    fdinfo              latency     mem          ns              pagemap       schedstat      stack     task             wchan
 attr          cmdline              cwd       gid_map             limits      mountinfo    numa_maps       personality   sessionid      stat      timens_offsets
 autogroup     comm                 environ   io                  loginuid    mounts       oom_adj         projid_map    setgroups      statm     timers
 auxv          coredump_filter      exe       ksm_merging_pages   map_files   mountstats   oom_score       root          smaps          status    timerslack_ns
 cgroup        cpu_resctrl_groups   fd        ksm_stat            maps        net          oom_score_adj   sched         smaps_rollup   syscall   uid_map
```

Here, we are looking at the process called `systemd` whose PID is 1. The first thing I love look in this folder is the `cmdline` file.

#### /proc/$PID/cmdline
```console
/proc/1
❯ cat -A cmdline
/sbin/init^@splash^@%
```
The command line shown in this file is cmd that had been used to invoke the executable. It also provides the arguments that had been used for the command.
> The `^@` character is a `NUL` in [caret notation](https://en.wikipedia.org/wiki/Null_character#Representation). It's shown in such a representation, because the C program invoking the `main(int argc, char** argv)` function expects the `argv` in NUL-terminated C-string array.

#### /proc/$PID/environ

The `environ` file describes the shell environment of the process. Here you'll find env variables that were set when the process was run. It could range from simple `$TERM` to sophisticated variables like `$PATH` etc.

```console
/proc/1
❯ cat environ
TERM=linux%                                                                                                                                                                                
```

#### Process memory

There are few other interesting files, but I'd like to focus more the memory of the process, as this is the crucial part that I've utilized extensively for building my project Kunai.

##### /proc/$PID/maps
This is the file that stores the memory mapping of the process. Let's have a look at it first.

```console
/proc/1
❯ sudo cat maps
55a4fd849000-55a4fd84f000 r--p 00000000 103:0a 6316636                   /usr/lib/systemd/systemd
55a4fd860000-55a4fd861000 rw-p 00017000 103:0a 6316636                   /usr/lib/systemd/systemd
55a4fdbba000-55a4fde74000 rw-p 00000000 00:00 0                          [heap]
7f5369074000-7f5369078000 r--p 00000000 103:0a 6316188                   /usr/lib/libelf-0.189.so
7f5369078000-7f536908a000 r-xp 00004000 103:0a 6316188                   /usr/lib/libelf-0.189.so
7f536908a000-7f536908e000 r--p 00016000 103:0a 6316188                   /usr/lib/libelf-0.189.so
...
7ffe66c98000-7ffe66d9a000 rw-p 00000000 00:00 0                          [stack]
7ffe66dc9000-7ffe66dcd000 r--p 00000000 00:00 0                          [vvar]
7ffe66dcd000-7ffe66dcf000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
```

The two lines I want you to focus here are 
```
55a4fdbba000-55a4fde74000 rw-p 00000000 00:00 0                          [heap]
```
```console
7ffe66c98000-7ffe66d9a000 rw-p 00000000 00:00 0                          [stack]
```
> Syntax: <br/>
> \<address start>-<address end>  \<mode>  \<offset>   \<major id:minor id>   \<inode id>   \<file path>  

These two lines contain the curx required to scan, edit, manipulate the process of a memory. Inherently, a process definitely has two kinds of memory, one is the \[stack] and the other is the \[heap[^heap]]. Now these two memory ranges can be read and manipulated, given that the user/process accessing the memory region has enough privileges. Kunai, in fact, utilizes these permissions to edit a processes' memory during it's runtime. It's like a [CheatEngine](https://en.wikipedia.org/wiki/Cheat_Engine) for linux!


The other lines present in the file are libraries or external entities that are managed by the same process but has dynamically loaded them outside the application point of view.

After getting the processes memory region from the `maps` file, it opens the `mem` file to edit or read the memory of that process.

#### /proc/$PID/mem
This the magic file! It contains the memory held by the process. You can access the memory, by seeking it to the location specified in the `maps` file.

For example, you can seek and edit memory as shown in the code below.
```python
def edit_mem():
    # ... A bit of preprocessing

    idx = mem.find(bytes(string_to_find, "ASCII")) # Finding the location of the string
    if idx == -1:
        eprint("[!] String not found in", mem.name)
        return
    print("[*] String found at:", idx, "in", mem.name)

    mem.seek(start + idx)                     # Seeking to the location of string
    mem.write(bytes(changed_string, "ASCII")) # Editing the string
    print(f"[*] Changed {original} to {changed} in {mem.name}")
```
You can refer to the file at [github](https://github.com/rseragon/Kunai/blob/3a883d06f40a5c645cb52b688a6555ad92c24e7d/scripts/mem_editor.py#L65)

Running a simple C program that allocates a string on the heap, some what similar to this code
```c
int main(void) {
  // String on the stack
  char s_stack[] = {'S', 'o', 'm', 'e', 'S', 't', 'r', 'i', 'n', 'g', '\0'};

  // String on the heap
  char *s_heap = strdup("HeapString");

  printf("Heap : %p\n", s_heap);  // Printing the location of string on stack
  printf("Stack: %p\n", s_stack); // Printing the location of string on heap 

  while (1) {
    sleep(5);
    printf("Stack: %s\n", s_stack);
    printf("Heap: %s\n", s_heap);
  }

  return 0;
}
```
When I run the C file and run the python file to edit the memory, you seen the end result as follows

```
❯ ./a.out
Heap : 0x55edc241d2a0
Stack: 0x7fffdc1daecd
Stack: SomeString
Heap: HeapString
Stack: SomeString
Heap: HeapString
Stack: SomeString
...
Heap: HeapString
Stack: SomeString
Heap: SheepString ===> The string changed!
...
```

```console
❯ sudo python mem_editor.py 585933 HeapString
String found at: 672
Change to: SheepString
Changed HeapString to SheepString
```

The "HeapString" is changed to "SheepString", as I've mentioned in the python script input. You can find the [mem_editor.py](https://github.com/rseragon/Kunai/blob/3a883d06f40a5c645cb52b688a6555ad92c24e7d/scripts/mem_editor.py) in github

There are a few other files present in `/proc/$PID`, but I'll leave that to your homework. Moving forward, I'll focus on more file inside the `/proc` directory.


## /proc/cpuinfo
This file contains information about the processor, such as its type, make, model, and performance.
```
/proc
❯ cat cpuinfo
processor	: 0                => Processor ID
vendor_id	: AuthenticAMD     => Manufacturer
cpu family	: 23
model		: 96
model name	: AMD Ryzen 5 4600H with Radeon Graphics
stepping	: 1
microcode	: 0x8600104
cpu MHz		: 3955.064
cache size	: 512 KB
physical id	: 0
siblings	: 12               => Total number of processors present along side this processor
core id		: 0             
cpu cores	: 6
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 16
wp		: yes
...
```

## /proc/meminfo
This contains information about memory usage, both physical and swap. Concatenating this file produces similar results to using 'free' or the first few lines of 'top'.
```
/proc
❯ cat meminfo
MemTotal:       15754988 kB
MemFree:         6072304 kB
MemAvailable:    6294144 kB
Buffers:           23472 kB
Cached:           505044 kB
SwapCached:       967288 kB
Active:          3207352 kB
...
```

## /proc/modules
Kernel modules currently loaded. Typically its output is the same as that given by the `lsmod` command.

```console
❯ lsmod
Module                  Size  Used by
uvcvideo              176128  0
videobuf2_vmalloc      20480  1 uvcvideo
uvc                    12288  1 uvcvideo
videobuf2_memops       16384  1 videobuf2_vmalloc
videobuf2_v4l2         40960  1 uvcvideo
videodev              389120  2 videobuf2_v4l2,uvcvideo
videobuf2_common       94208  4 videobuf2_vmalloc,videobuf2_v4l2,uvcvideo,videobuf2_memops
mc                     90112  4 videodev,videobuf2_v4l2,uvcvideo,videobuf2_common
...
```
```console
❯ cat /proc/modules
uvcvideo 176128 0 - Live 0x0000000000000000
videobuf2_vmalloc 20480 1 uvcvideo, Live 0x0000000000000000
uvc 12288 1 uvcvideo, Live 0x0000000000000000
videobuf2_memops 16384 1 videobuf2_vmalloc, Live 0x0000000000000000
videobuf2_v4l2 40960 1 uvcvideo, Live 0x0000000000000000
videodev 389120 2 uvcvideo,videobuf2_v4l2, Live 0x0000000000000000
videobuf2_common 94208 4 uvcvideo,videobuf2_vmalloc,videobuf2_memops,videobuf2_v4l2, Live 0x0000000000000000
...
```


## /proc/mounts
Lists all the mounted file systems and their parameters. This is similar to the file in `/etc/mtab`
```console
❯ cat /etc/mtab
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
sys /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
dev /dev devtmpfs rw,nosuid,relatime,size=7856308k,nr_inodes=1964077,mode=755,inode64 0 0
run /run tmpfs rw,nosuid,nodev,relatime,mode=755,inode64 0 0
efivarfs /sys/firmware/efi/efivars efivarfs rw,nosuid,nodev,noexec,relatime 0 0
...
```
```console
❯ cat /proc/mounts
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
sys /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
dev /dev devtmpfs rw,nosuid,relatime,size=7856308k,nr_inodes=1964077,mode=755,inode64 0 0
run /run tmpfs rw,nosuid,nodev,relatime,mode=755,inode64 0 0
efivarfs /sys/firmware/efi/efivars efivarfs rw,nosuid,nodev,noexec,relatime 0 0
...
```


## /proc/uptime
The output of the command `uptime` is derived from this file. You can verify it from this [line](https://github.com/coreutils/coreutils/blob/28b176085f04a6227d7fadd28a129b5cb02dfbf5/src/uptime.c#L194) of coreutils library
```
/proc
❯ cat uptime
2711524.13 590084.11

/proc
❯ uptime
 23:06:08 up 31 days,  9:12,  1 user,  load average: 2.42, 2.61, 2.47
```

## /proc/version
The contents of this file are similar to the output of the command `uname`

```console
/proc
❯ cat version
Linux version 6.5.8-arch1-1 (linux@archlinux) (gcc (GCC) 13.2.1 20230801, GNU ld (GNU Binutils) 2.41.0) #1 SMP PREEMPT_DYNAMIC Thu, 19 Oct 2023 22:52:14 +0000

/proc
❯ uname -a
Linux yoru 6.5.8-arch1-1 #1 SMP PREEMPT_DYNAMIC Thu, 19 Oct 2023 22:52:14 +0000 x86_64 GNU/Linux
```

## /proc/swaps
Similar to `swapon -s` command
```console
/proc
❯ cat swaps
Filename				Type		Size		Used		Priority
/dev/nvme0n1p3                          partition	16777212	3821596		-2

/proc
❯ swapon -s
Filename				Type		Size		Used		Priority
/dev/nvme0n1p3                          partition	16777212	3800604		-2
```

## /proc/sys/
Taken from tldp
> This is not only a source of information, it also allows you to change parameters within the kernel without the need for recompilation or even a system reboot. Take care when attempting this as it can both optimize your system and also crash it. To change a value, simply echo the new value into the file. 

An example is given below in the section on the file system data. You can create your own boot script to perform this every time your system boots.

For example, to enable coredumps for processes in linux, you can do the following.
```
ulimit -c unlimited
echo 1 > /proc/sys/kernel/core_uses_pid
```
Refer [this](https://access.redhat.com/solutions/4896) link for more information.

## /proc/self/
This is one of the most interesting directory in procfs. Why? The answer is quite simple. It represents your current running process. 
```console
❯ ls -l /proc | grep 'self'
lrwxrwxrwx    - root              5 Dec  2024 self -> 615669
```
As you can see, the self directory is mapped to another PID directory in the procfs. It's a soft link that is different for every process and is assigned by the kernel.


# References
- [/proc from tldp](https://tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)


# Footnotes
[^heap]: There are few processes that don't allocate heap space. But for this example, let's just consider they do exist.
