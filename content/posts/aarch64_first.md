---
title: AArch64 assembly - part 1
date: 2022-02-02T10:58:00+01:00
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
0. [Home](README.md)</br>
1. [Let's break the ice](FIRST.md)</br>
2. [Under the surface](SECOND.md)</br>
3. [Under the microscope](THIRD.md)</br>
4. [Cross-compilation](FOURTH.md)</br>
5. [Resources](RESOURCES.md)</br>

----

## Let's break the ice
... or, let's quickly generate some assembly code and run it.</br>

### Our very first program
Let's write a small assembly program that does nothing but exits by leaving a specific *return code*.</br>
Copy and paste the following code into a file named `answer.s`.

<pre>
/*
  File: answer.s
  Purpose: runs and, immediately, exits while leaving a return code that can be displayed
           through the OS
  Example:
    $ ./answer
    $ echo $?
    42
*/
.text

.global _start

_start:
    mov x0, #42  // x0 ←  42
    mov x8, #93  // x8 ←  93 (__NR_exit)
    svc #0       // syscall
</pre>

The program can be assembled and linked with `as` and `ld` from `binutils`:

<pre>
$ <b>as -g answer.s -o answer.o</b>

$ <b>ld answer.o -o answer</b>
</pre>

**NOTE**: option `-g` in the first step is used to retain symbols in the code. It is optional and only used for the purpose of these notes.</br>
</br>
Finally, as mentioned in the code, it can be run as follows:

<pre>
$ <b>./answer</b>

$ <b>echo $?</b>
42
</pre>

### Wut?
Oh! Wow! Indeed it works... and, apparently, it didn't break anything. Good!</br>
Let's dig a bit deeper and look at what we've done.</br>
</br>
Our file `answer.s` is a tiny text file abiding to a specific syntax. A few elements can be identified:
- `/*` and `*/`: multi-line comment, anything in between is ignored by the assembler (`as`)
- `//`: single-line comment, anything between the double forward slashes and the end of the line is ignored
- `.text`: this statement marks the beginning of our code
- `.global`: this statemenet is followed by a symbol (`_start`) which is *exposed* to the linker (`ld`); anything after `_start` is our *real* code
- `mov`: this is an AArch64 instruction whose aim is to move some data (`#42`, decimal `42`) to `x0` (the first general-purpose register in ARMv8 processors)
- `svc`: this instruction passes control back to the OS while invoking the `syscall` whose number has been stored in `x8`
</br>
Once the file has been saved it is assembled with `as` and, finally, linked with `ld`. The resulting file can be run from the command line and its output can be read with the `echo $?` command.</br>
Easy, huh!?!</br>

**NOTE**: for more info on `syscall` try `man syscall`. Hint:

<pre>
$ grep 93 /usr/include/asm-generic/unistd.h | head -n1
#define __NR_exit 93
</pre>

