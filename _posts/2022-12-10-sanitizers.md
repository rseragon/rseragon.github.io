---
title: Sanitizers 🤝
date: 2022-12-19 01:00:03 +0530
categories: [C/C++, advanced]
tags: [Sanitizers, C, Cpp]
---

Did you know that nearly 80% of illness-causing germs are spread by your hands!?<br/>
One of the most proven methods that ensure the destruction of **99.9%** of these 
illness-causing germs is using a Dettol hand sanitizer!<br/>
This hand sanitizer had been incorporated into millions of lives ever since humanity
started fearing the pandemic (*cough* *cough* COVID *cough* *cough*).

Well, whether the sanitizers we would be talking about could catch up **99.9%** of the bugs is a controversial
matter, but most of the daily-driver based projects which have used them turned out
to have have some of undetected bugs which surfaced with the help of these Sanitizers

> Though this tutorial uses Clang as its default compiler, most of the sanitizers
> have also been implemented for GCC(except for a few). So, feel free to use whichever compiler you
> prefer.
{: .prompt-tip}

# TL;DR
> Sanitizers are run-time/dynamic binary analyzers, which soak into the binary during compilation
> and trigger when a certain flag or a condition is violated. These are very helpful in the detection
> of elusive bugs which are hard to find. They are implemented thoroughly in Clang and GCC(pretty much)
> and are considered to pretty stable.
{: .prompt-info}
Few of the sanitizers are


|Name|Detects|
|:----:|:-----:|
|Address Sanitizer|Out of bounds access, Use after free, double free...|
|Memory Sanitizer|Uninitalized reads|
|Leak Sanitizer|Run-time memory leaks|
|Undefined Behavior Sanitizer|Undefined Behavior|
|Thread Sanitizer|Data races and dead locks|


# Sanitizers in C/C++

Memory-unsafe languages like C and C++ require some kind of support for the detection
of bugs that are hidden from a normal developer's eye. Sanitizers take this supportive role
by implanting themselves into the binary and constantly looking for bugs in the execution.
There are many sanitizers specified by google, you can refer to it [here](https://github.com/google/sanitizers)

Now, without further ado, let's get started :D

## Address sanitizer
Popularly knows as ASan, it's used to detect *address/memory access errors*. 
Let's start with an example :D

```c++
int main() {
  int *arr = new int[4]; // Allocating 4 ints 

  // Oh, no!
  // Accessing array out of bounds.
  // This is subtle but a very common rookie mistake
  arr[4] = 100;

  return 0;
}

```
{: file="outofbounds.cpp"}

Compiler output (Skip this output, read the explanation below it, and if you feel comfortable come back here!)
```console
=================================================================
==903795==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000020 at pc 0x5564f9892dfe bp 0x7ffce60719d0 sp 0x7ffce60719c8
WRITE of size 4 at 0x602000000020 thread T0
    #0 0x5564f9892dfd in main /programs/outofbounds.cpp:6:10
    #1 0x7fb07101c28f  (/usr/lib/libc.so.6+0x2328f) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)
    #2 0x7fb07101c349 in __libc_start_main (/usr/lib/libc.so.6+0x23349) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)
    #3 0x5564f97970d4 in _start /build/glibc/src/glibc/csu/../sysdeps/x86_64/start.S:115

0x602000000020 is located 0 bytes to the right of 16-byte region [0x602000000010,0x602000000020)
allocated by thread T0 here:
    #0 0x5564f9890052 in operator new[](unsigned long) (/programs/a.out+0x11a052) (BuildId: 4d56e849fbab3be690023f3f7405b335c8676ce4)
    #1 0x5564f9892db8 in main /programs/outofbounds.cpp:4:14
    #2 0x7fb07101c28f  (/usr/lib/libc.so.6+0x2328f) (BuildId: 1e94beb079e278ac4f2c8bce1f53091548ea1584)

SUMMARY: AddressSanitizer: heap-buffer-overflow /programs/outofbounds.cpp:6:10 in main
Shadow bytes around the buggy address:
  0x0c047fff7fb0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7fc0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7fd0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7fe0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c047fff7ff0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c047fff8000: fa fa 00 00[fa]fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8010: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8020: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8030: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8040: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff8050: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==903795==ABORTING
```
Yeah, I get it. The output is daunting; Getting bombarded with too much information
does get us a bit irky. But, you don't have to fully follow it to understand it. All you
have to do is break the info into parts and consume it selectively (at least that's what I do).

+ Look the line with the word **SUMMARY** written on it:<br/>
`SUMMARY: AddressSanitizer: heap-buffer-overflow /programs/outofbounds.cpp:6:9 in main`<br/>
This basically explains everything we need to know about the error!

In short, there's been a heap overflow and if you look at the first few lines of the output message 
we can find `WRITE of size 4 at 0x602000000020 thread T0`, i.e. we've written 4 bytes to the memory
address `0x602...` which was out of our scope (Now read the output, if you've skipped it).

![out-of-bounds](https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/sanitizers/anim/outofbounds.gif)

The sanitizer got to know about this because it has *poisoned* the memory around the heap
which lets it know if there has been any overflowing or invalid access of the memory.

If you look again at the output, you see quite a diagram drawn out of how the memory is been
layed out and what area is accessible and what is not. This output itself is quite intuitive
and is just made to give us a high level pretty printed perspective of our memory. So, I don't 
think it needs any special explanation.

## Memory Sanitizer
This is very much similar to ASan, but detects different bugs compared to it. 
One of the main agenda of this is to detect "uninitalized reads", in short
it looks for non inited vars/memory, *innit mate?*

Consider this example, (Though this program doesn't make any sense, just roll along with it)<br/>
**NOTE:** This sample only works in clang(get version info)
```c++
#include <vector>
#include <iostream>
int main(int argc, char** argv) {

  // Create a vector with 10 block uninited data
  std::vector<int> vec{10};

  // Accessing the uninited data 
  // this results in garbage value
  std::cout << vec[argc] << '\n';

  // ...
}
```
Now, if we look at the output
```console
❯ clang++ uninited_data.cpp -fsanitize=memory -g && ./a.out
Uninitialized bytes in MemcmpInterceptorCommon at offset 144 inside [0x7ffc188a4660, 256)
==978373==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x562d29e5b2dd in __interceptor_memcmp (/programs/a.out+0x9e2dd) (BuildId: a7afa8f61f81cd90f16a8dc3ceb32b0b5b94c1c7)
	...
    #7 0x562d29e70bb3 in main /programs/uninited_data.cpp:5:12
	...

SUMMARY: MemorySanitizer: use-of-uninitialized-value (/programs/a.out+0x9e2dd) (BuildId: a7afa8f61f81cd90f16a8dc3ceb32b0b5b94c1c7) in __interceptor_memcmp
Exiting
```
(This time I've omitted unwanted data, so we can only focus on the important parts)

From this we can see that, if we try to access any data which is not properly initialized, then
it's helpful for us if we get this error because sometimes we might just work on these garbage
values which might not get our results as intended (acessing those values is practially an UB).
So MSan helps us to find this unforseen accesses and reduce errors in our program.


Though MSan and ASan are very similar, the difference between them is that ASan will not
trigger an error until the address on which the operation is operating on is *valid* i.e. in
the range allocated for the program. While MSan works on the intricate details of how the
allocated memory is accessed and tries to detect the uninited data so that we don't work
with garbage values.


## Thread sanitizer
This is one of the most useful sanitizer as it helps us to detect "data races". 
> Data race: Occurs when multiple threads access the same memory location without proper synchronization.
{: .prompt-info }

![data-race](https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/sanitizers/anim/datarace.gif)

Data races are very unpredictable as they are dependent on the system scheduler to decide when the process
can access the data. They show up unexpectedly and many developers consider them to be very tricky
to deal with.

Thread sanitizer is a savior if you are writing a multi-threaded program. Let's understand
its working with a small example.

```c++
#include <iostream>
#include <thread>

// Declare globally accessible variable
int GLOBAL = 0;

void update_global() {
	// Access it via spawned thread
	GLOBAL++;	
	std::cout << "From thread: " << GLOBAL << '\n';
}

int main() {
	std::thread t{update_global};

	// Accessing it via main thread
	GLOBAL++;

	std::cout << "From main: " << GLOBAL << '\n';
	t.join();

	std::cout << "Final: " << GLOBAL << '\n';
	return 0;
}
```
Output of it without any sanitizer
Try 1
```console
❯ clang++ data_race.cpp && ./a.out
From main: 1
From thread: 2
Final: 2
```
Try 18
```console
❯ clang++ data_race.cpp && ./a.out
From main: 2
From thread: 2
Final: 2
```

As you can see even though it's a bit subtle, there is ambiguity in the working of the program.
This might lead to unwanted experiences during run time. Most of the time, these types of bugs
go unnoticed, so here we can use a Thread Sanitizer to identify the problem.
This ambiguity here is called "Data race" and is a pretty hefty concept in computer science.


Output with thread sanitizer:
```console
❯ clang++ data_race.cpp -fsanitize=thread && ./a.out
From thread: 1
==================
WARNING: ThreadSanitizer: data race (pid=1091036)
  Write of size 4 at 0x55b49fbbf7cc by main thread:
    #0 main <null> (a.out+0xe9781) (BuildId: d27991fc397f0f737ca49849f28ef74771cb1a63)

  Previous write of size 4 at 0x55b49fbbf7cc by thread T1:
    #0 update_global() <null> (a.out+0xe96a8) (BuildId: d27991fc397f0f737ca49849f28ef74771cb1a63)
	....
    #6 execute_native_thread_routine /usr/src/debug/gcc/libstdc++-v3/src/c++11/thread.cc:82:18 (libstdc++.so.6+0xd62f2) (BuildId: 735a3d0cc7699fd69337361cba4aedb644b2a7ed)

  Location is global 'GLOBAL' of size 4 at 0x55b49fbbf7cc (a.out+0x15157cc)

  Thread T1 (tid=1091038, finished) created by main thread at:
    #0 pthread_create <null> (a.out+0x682d6) (BuildId: d27991fc397f0f737ca49849f28ef74771cb1a63)
	....
    #3 main <null> (a.out+0xe9764) (BuildId: d27991fc397f0f737ca49849f28ef74771cb1a63)

SUMMARY: ThreadSanitizer: data race (/programs/a.out+0xe9781) (BuildId: d27991fc397f0f737ca49849f28ef74771cb1a63) in main
==================
From main: 2
Final: 2
ThreadSanitizer: reported 1 warnings
```

As usual, I've removed some useless data in it, to make it more readable.


From the sanitizer's output, we can figure out that there has been a data race between two threads trying to access 
the global variable `GLOBAL`. Usually, this might go unnoticed, but TSan is powerful enough to 
identify these nitty-witty bugs.

## Other sanitizers
There are few other sanitizers which are interesting, follow the links if you wanna look them up.
- [Leak Sanitizer](https://clang.llvm.org/docs/LeakSanitizer.html): Used for detecting memory leaks
- [UB Sanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html): Finding Undefined behavior in the program
You can refer to more informational docs [here](https://github.com/google/sanitizers)


# Alternatives
[Valgrind](https://valgrind.org/) is one of the most prominent alternative for sanitizers, but sanitizers have proven
to be way too much faster compared to valgrind. Many projects have started to shift to in-built
sanitizers, but even then valgrind is considered to be pretty stable, so it's
not a choice that can be neglected.

# Refernces
- [Google Sanitizers](https://github.com/google/sanitizers)
- [Redhat blog](https://developers.redhat.com/blog/2021/05/05/memory-error-checking-in-c-and-c-comparing-sanitizers-and-valgrind)
- [Trail of bits blog](https://blog.trailofbits.com/2019/06/25/creating-an-llvm-sanitizer-from-hopes-and-dreams/)
- [Clang docs](https://clang.llvm.org/docs/index.html)

