---
title: Installing Stable Diffusion web UI on AMD Radeon RX 5500M
date: 2023-12-04 10:00:03 +0530
categories: [utility]
tags: [ROCm, AMD]
image: https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/rocm-navi14/background.png
---

> This tutorial has been tested on AMD RX 5500M gpu and is working properly.
{: .prompt-info}

## Prerequisites
- Linux distro: Arch
- Hardware
  + AMD Radeon RX 5500M or any Navi 14 based GPU
- Lots of patience!
- AUR package manager: `yay`, `paru` etc.

## Installing Software
### 1. ROCm libraries and dependencies 

  > NOTE: This installation is gonna take a lot of time
  {: .prompt-tip}

  ```bash
  yay -S rocm-hip-sdk rocm-opencl-sdk rocminfo rocm-smi-lib
  ```
  > Replace `yay` with your favorite AUR package manager
  {: .prompt-info}

  - ROCm HIP:    A Langauge similar to CUDA by Nvidia.
  - ROCm OpenCL: Useful for enchacing run time on cpu based models like (GGFU/GGML)
  - rocminfo:    To verify if ROCm is working.
  - rocm-smi:    An efficient way of looking at GPU statistics


  > Failed to install? run the following command below
  ```bash
  HSA_OVERRIDE_GFX_VERSION=10.3.0 yay -S rocm-hip-sdk rocm-opencl-sdk rocminfo rocm-smi-lib
  ```
  This makes the installation compatible to gfx1030 which for some reason also works on our device[^github-rocm-scripts]

### 1.1 Add few environment variables
Add these two in your `.bashrc` or `.zshrc`
```
export HSA_OVERRIDE_GFX_VERSION=10.3.0
PATH=$PATH:/opt/rocm/bin
```

### 2. Verify ROCm installation
  ```bash
  /opt/rocm/bin/rocminfo
  ```

  This should output some long information like the below output

  ```
  ROCk module is loaded
  =====================
  HSA System Attributes
  =====================
  Runtime Version:         1.1
  System Timestamp Freq.:  1000.000000MHz
  Sig. Max Wait Duration:  18446744073709551615 (0xFFFFFFFFFFFFFFFF) (timestamp count)
  Machine Model:           LARGE
  System Endianness:       LITTLE

  ========== HSA Agents
  ==========

  ....


  *******
  Agent 2
  *******
    Name:                    gfx1012
    Uuid:                    GPU-XX
    Marketing Name:          AMD Radeon RX 5500M
    Vendor Name:             AMD
    Feature:                 KERNEL_DISPATCH
    Profile:                 BASE_PROFILE
    Float Round Mode:        NEAR

  ...

  ```
  If you found something like this with your GPU name correctly displayed, then **ROCm is installed!**

### 3. Installing PyTorch
First, check if your cpu supports **AVX2**[^avx2-check].
```bash
grep avx2 /proc/cpuinfo
```
If there is **no** output then your CPU doesn't support AVX2, so **skip** the next step.

#### Install Optimized AVX2 Pytorch
```
pacman -S python-pytorch-opt-rocm
```

#### Install for normal PyTorch (For non AVX2 CPUs)
```
pacman -S python-pytorch-rocm
```

#### Install torch vision
```bash
yay -S python-torchvision-rocm
```

### 4. Verify PyTorch installation
```bash
python -c 'import torch; print(torch.__version__)'
```
If this command fails then you would need to reinstall PyTorch again (Follow the previous step).

### 5. Installing Stable Diffusion UI
#### Clone the repo
```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```
#### Setup `venv`
Copy-pasta'd from [Stable diffusion webui wiki](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Install-and-Run-on-AMD-GPUs#setup-venv-environment)
```bash
python -m venv venv --system-site-packages
source venv/bin/activate
pip install -r requirements.txt
```

### Run the UI
```bash
source venv/bin/activate
./webui.sh --precision full --no-half --upcast-sampling --opt-sub-quad-attention --lowvram --disable-nan-check
```
The above command is a result of me fixing multiple errors which I've faced on my Radeon RX 5500M, feel free to experiment it yourself.

> Place your safetensors in `models/Stable-diffusion/`
{: .prompt-tip}

# Results!
![img1](https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/rocm-navi14/00002-1693224475.png)
![img2](https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/rocm-navi14/00003-1693224476.png)
![img3](https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/rocm-navi14/00004-1693224477.png)

Model used: [ZHMix-Dramatic](https://civitai.com/models/148158/zhmix-dramatic?modelVersionId=234270)

> Note: All of these images are generated and I have no rights or licenses over these images

## Some useful links

[Hip out of memory error](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/6460)<br/>
[GPU Specs](https://www.techpowerup.com/gpu-specs/radeon-rx-5500m.c3460)<br/>
[RX Series information](https://en.wikipedia.org/wiki/Radeon_RX_5000_series#Mobile)<br/>
[Software support list](https://en.wikipedia.org/wiki/List_of_AMD_graphics_processing_units#Features_overview)<br/>

## References 
[^github-rocm-scripts]: Refer this [repo](https://github.com/xuhuisheng/rocm-build/tree/master/navi14)
[^avx2-check]: [stackoverflow](https://stackoverflow.com/questions/37480071/how-to-tell-if-a-linux-machine-supports-avx-avx2-instructions)
