---
title: AArch64 assembly - part 2
date: 2022-02-02T11:05:00+01:00
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

##  Under the surface
In the process we've followed not only we've generated file `answer.s` but `answer.o` and `answer` as well. You may be wondering what are those?</br>
`answer.o` is the object-file, it complies with the ELF format (`man elf`). Its contents can be displayed through the `objdump` command.

```
$ <b>file answer.o</b>
answer.o: ELF 64-bit LSB relocatable, ARM aarch64, version 1 (SYSV), with debug_info, not stripped

$ <b>objdump -D answer.o</b>
answer.o:     file format elf64-littleaarch64

Disassembly of section .text:

0000000000000000 <_start>:
   0:	d2800540 	mov	x0, #0x2a                  	// #42
   4:	d2800ba8 	mov	x8, #0x5d                  	// #93
   8:	d4000001 	svc	#0x0
...
```

File `answer` instead, is an ELF **executable**. `readelf` shows detailed information on its contents.

```
$ <b>file answer</b>
answer: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, with debug_info, not stripped

$ <b>readelf -a answer</b>
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           AArch64
...

Symbol table '.symtab' contains 12 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000400078     0 SECTION LOCAL  DEFAULT    1
     2: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS answer.o
     3: 0000000000400078     0 NOTYPE  LOCAL  DEFAULT    1 $x
     4: 0000000000410084     0 NOTYPE  GLOBAL DEFAULT    1 _bss_end__
     5: 0000000000410084     0 NOTYPE  GLOBAL DEFAULT    1 __bss_start__
     6: 0000000000410084     0 NOTYPE  GLOBAL DEFAULT    1 __bss_end__
     7: 0000000000400078     0 NOTYPE  GLOBAL DEFAULT    1 _start
     8: 0000000000410084     0 NOTYPE  GLOBAL DEFAULT    1 __bss_start
     9: 0000000000410088     0 NOTYPE  GLOBAL DEFAULT    1 __end__
    10: 0000000000410084     0 NOTYPE  GLOBAL DEFAULT    1 _edata
    11: 0000000000410088     0 NOTYPE  GLOBAL DEFAULT    1 _end

No version information found in this file.
```

**NOTE**: both files contain `debug_info`, this will be useful when running the executable through `gdb`.

