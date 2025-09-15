---
slug: linuxfromscratch
date: 2025-09-14T11:46:00+03:00
# lastmod: 2025-09-14T11:46:00+03:00 # Last modification date
tags:
- AWS
categories:
- Cloud
title: Building a linux system from scratch
---

As an experiment in deepening my understanding of Linux internals, I took on the challenge of working through **Linux From Scratch (LFS)**. Unlike installing a distribution like Ubuntu or Fedora, LFS walks you through the entire process of building a Linux system from the ground up, starting with nothing but a host Linux environment and source code. Here’s what I learned along the way.

## Preparing the Host Environment
The first step was setting up a suitable host system. Since LFS requires a working Linux environment with GCC, Binutils, and other build tools, I spun up a clean Ubuntu VM.  

Tasks included:
- Installing required development tools (`build-essential`, `bison`, `gawk`, etc.).  
- Creating a dedicated **partition** (or loopback image file) to serve as the target for the new LFS system.  
- Setting up a mount point (e.g., `/mnt/lfs`) for building the system in isolation.  

## Building the Toolchain
The heart of LFS is constructing a **temporary toolchain**—a minimal set of compiler tools that are independent from the host system. This involved compiling:
- **Binutils** (assembler, linker).  
- **GCC** (cross-compiler tuned for the target).  
- **Glibc** (the standard C library).  

The toolchain was installed into `/mnt/lfs/tools` and used for all subsequent builds, ensuring the final system was self-contained.

## Compiling the Base System
With the toolchain ready, I moved on to building the core components of the new system:
- **Coreutils** (basic commands like `ls`, `cp`, `rm`).  
- **Bash** (the shell).  
- **Grep, Sed, Findutils, Make, Diffutils**, and other essential utilities.  

Each package was built from source, tested, and installed into the LFS root. This part of the process was repetitive but illuminating—every tool we take for granted had to be deliberately built and installed.

## Building and Installing the Linux Kernel
One of the most exciting steps was compiling the Linux kernel itself:
1. Downloaded the kernel source.  
2. Configured it with `make menuconfig`, keeping things minimal while ensuring support for my hardware.  
3. Compiled and installed both the kernel and its modules into `/boot`.  

This was the point where the project began to feel like a real operating system.

## Setting Up Boot and Init
To make the system bootable, I configured:
- **Grub** as the bootloader.  
- A minimal **/etc/fstab** file to describe the root partition.  
- A simple **init system** using SysV init scripts (LFS sticks to basics, though more advanced init systems can be added later).  

Once complete, the system could boot independently into a shell prompt.

## The First Boot
After unmounting and rebooting into the new partition, I was greeted by something small but meaningful:  
```text
Welcome to your new Linux system!
No desktop, no package manager, no extras—just a bare but functional Linux system I had built by hand.
```


## Reflections and Learnings

- Dependencies make sense: Building each component clarified how tools rely on one another.
- The kernel isn’t everything: A working Linux system is the sum of many small utilities layered on top of the kernel.
- Minimal ≠ unusable: Even without a GUI or package manager, the system could compile software, run scripts, and be extended further.

## Final Thoughts

Working through LFS was a demanding but rewarding exercise. It gave me a clear view of how Linux systems are assembled, from compiler toolchains to bootloaders. The end result was a minimal but functional Linux OS that I understood from the ground up—because I built it, one package at a time.
