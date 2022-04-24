---
title: AArch64 assembly - part 4
date: 2022-02-02T11:08:00+01:00
draft: false
description: A short series of articles re. Assembly on ARM64
#series: AArch64
tags:
- assembly
- arm64
- aarch64
---

## AArch64

### Index
0. [Home]({{< ref "/posts/aarch64_home" >}})</br>
1. [Let's break the ice]({{< ref "/posts/aarch64_first" >}})</br>
2. [Under the surface]({{< ref "/posts/aarch64_second" >}})</br>
3. [Under the microscope]({{< ref "/posts/aarch64_third" >}})</br>
4. [Cross-compilation]({{< ref "/posts/aarch64_fourth" >}})</br>
5. [Resources]({{< ref "/posts/aarch64_resources" >}})</br>

----

## Cross-compilation
Interestingly, one does not need to own an ARM64 processor. With the help of QEMU user mode emulation (`qemu-user`) and the GNU C compiler for AArch64 (`gcc-aarch64-linux-gnu`), assembling and linking native code is a breeze.</br>
Starting from exactly the source code let's run:
```
$ <b>aarch64-linux-gnu-as answer.s</b>

$ <b>aarch64-linux-gnu-gcc -static -o answer a.out</b>

$ <b>./answer</b>

$ <b>echo $?</b>
42
```

Of course, **no tricks**, the output above has been collected on:
```
$ <b>uname -m</b>
x86_64

$ <b>file a.out</b>
a.out: ELF 64-bit LSB relocatable, ARM aarch64, version 1 (SYSV), not stripped

$ <b>file answer</b>
answer: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped
```

