---
title: AArch64 assembly - part 0 
date: 2022-01-24T14:41:00+01:00
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

### Background
I'm **not** exactly a programmer. I can write some (mostly bad) code, understand a few tiny bits here and there, but I can't claim to be an expert here.</br>
Be that as it may, I'm curious, I'm an avid learner, I sweat my way through new things until I reach that A-HA moment.</br>
I love tutorials on the Internet because, that's my hope, one can quickly be projected into the practical parts of a new topic. Digging deeper should be left as an exercise to the reader.</br>
</br>
_Clerk: You've got a pet halibut?_</br>
_Man: Yes, I chose him out of thousands. I didn't like the others, they were all too flat._</br>
</br>
However, I have struggled to find the **perfect** tutorial. This one is too detailed, this one is too specific... I'm being picky, I know, but the impression I get is that assembly is somehow for the initiated.</br>
BTW, some documents on the Internet are exceptionally good. You may want to take a look at the [Resources](aarch64_resources) page.</br>
</br>
After reading a few very good ones, I have taken the bold move to write my own. I've done that with the intention to leave some notes to myself that would match my own learning process. I'd be glad if the same notes are helpful to others.</br>
That being said, let's start and feel free to ping me or open some _meaningful_(1) PR in case you'd like to contribute.</br>

**NOTE**: The following outputs have been generated on a Raspberry Pi 4.
Here's what my environment looks like:</br>

- board
```
$ lscpu
Architecture:                    <b>aarch64</b>
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
CPU(s):                          4
On-line CPU(s) list:             0-3
Thread(s) per core:              1
Core(s) per socket:              4
Socket(s):                       1
Vendor ID:                       <b>ARM</b> 
Model:                           3
Model name:                      <b>Cortex-A72</b>
...
```

- software
```
$ as --version
GNU assembler (GNU Binutils for Debian) 2.35.2
...

$ ld --version
GNU ld (GNU Binutils for Debian) 2.35.2
...

$ gdb --version
GNU gdb (Debian 10.1-1.7) 10.1.90.20210103-git
...
```

----

(1) meaningful: such as blatant errors or significant improvements. Syntax errors are also welcome but they are only accepted in multiples of three :)

