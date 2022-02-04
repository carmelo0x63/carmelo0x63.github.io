---
title: AArch64 assembly - part 3
date: 2022-02-02T11:07:00+01:00
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

##  Under the microscope
We could be happy with the overall result but, since we're dealing with a very low-level language, this little intro wouldn't be complete if `GDB` wasn't mentioned here.</br>
`GDB` is the GNU Debugger and, while it is indeed full of features and almost an OS by itself, it is not as scary as one could imagine. It is truly the microscope with which we can see what our tiny program is doing in real-time.</br>
Let's run our executable through `gdb`. Remember, the file must have been assembled with option `-g` enabled!</br>

<pre>
$ <b>gdb answer</b>
GNU gdb (Debian 10.1-1.7) 10.1.90.20210103-git
...
Reading symbols from answer...
(gdb)
</pre>

Before doing anything else, let's associate a **breakpoint** with the symbol `_start` (see the [source code](answer.s) in case you need to refresh your memory).

<pre>
(gdb) <b>break _start</b>
Breakpoint 1 at 0x400078: file answer.s, line 15.
</pre>

**NOTE**: a *breakpoint* stops the execution of a program at a given point. From there the execution can be controlled step-by-step for instance.</br>
Finally, we `run` our program as follows:

<pre>
(gdb) <b>run</b>
Starting program: /.../answer

Breakpoint 1, _start () at answer.s:15
15	    mov x0, #42  // x0 ←  42
</pre>

The output shows that execution has stopped at `Breakpoint 1`, corresponding to label `_start` and that the next step will be running the code shown below. `15` represents the row number in the source code.</br>
Taking a look at the registers will show that their contents has been reset and nothing has been changed yet.

<pre>
(gdb) <b>info registers</b>
<b>x0             0x0                 0</b>
x1             0x0                 0
x2             0x0                 0
x3             0x0                 0
x4             0x0                 0
x5             0x0                 0
x6             0x0                 0
x7             0x0                 0
<b>x8             0x0                 0</b>
x9             0x0                 0
...
pc             0x400078            0x400078 <_start>
...
</pre>

To move forward we need to tell `gdb` to run the next instruction: `mov x0, #42`.

<pre>
(gdb) <b>next</b>
16	    mov x8, #93  // x8 ←  93 (__NR_exit)
</pre>

Again, it'll execute our command then stop showing the following step.
Not unexpectedly, by examining the registers we'll show that something has indeed changed:

<pre>
(gdb) <b>info registers</b>
<b>x0             0x2a                42</b>
x1             0x0                 0
x2             0x0                 0
x3             0x0                 0
x4             0x0                 0
x5             0x0                 0
x6             0x0                 0
x7             0x0                 0
<b>x8             0x0                 0</b>
x9             0x0                 0
...
pc             0x40007c            0x40007c <_start+4>
...
</pre>

Specifically, `x0` now contains the value `0x2a` (corresponding to decimal `42`). Likewise `pc`, the **Program Counter**, displays the address of the next instruction in memory, or `_start` = 0x400078 + 4 bytes = 0x40007c.</br>Once again we type `next` to run `mov x8, #93`. `gdb` will execute the instruction and immediately stop.

<pre>
(gdb) <b>next</b>
17	    svc #0       // syscall
</pre>

The register status has been updated with new data (`0x5d` in register `x8`) and `pc` has been incremented.

<pre>
(gdb) <b>info registers</b>
<b>x0             0x2a                42</b>
x1             0x0                 0
x2             0x0                 0
x3             0x0                 0
x4             0x0                 0
x5             0x0                 0
x6             0x0                 0
x7             0x0                 0
<b>x8             0x5d                93</b>
x9             0x0                 0
...
pc             0x400080            0x400080 <_start+8>
...
</pre>

Finally, by typing `next` one more time, the last instruction (`svc #0`) will be run and execution will stop.

<pre>
(gdb) <b>next</b>
[Inferior 1 (process 69014) exited with code 052]
</pre>

### GDB help
`gdb` offers a help command to investigate its many features. The Internet has many tutorials and a number of good books exist.

<pre>
(gdb) <b>help</b>
List of classes of commands:
aliases -- User-defined aliases of other commands.
breakpoints -- Making program stop at certain points.
data -- Examining data.
files -- Specifying and examining files.
internals -- Maintenance commands.
obscure -- Obscure features.
running -- Running the program.
stack -- Examining the stack.
status -- Status inquiries.
support -- Support facilities.
text-user-interface -- TUI is the GDB text based interface.
tracepoints -- Tracing of program execution without stopping the program.
user-defined -- User-defined commands.
Type "help" followed by a class name for a list of commands in that class.
Type "help all" for the list of all commands.
Type "help" followed by command name for full documentation.
Type "apropos word" to search for commands related to "word".
Type "apropos -v word" for full documentation of commands related to "word".
Command name abbreviations are allowed if unambiguous.


(gdb) <b>help running</b>
Running the program.
List of commands:
advance -- Continue the program up to the given location (same form as args for break command).
attach -- Attach to a process or file outside of GDB.
continue, fg, c -- Continue program being debugged, after signal or breakpoint.
detach -- Detach a process or file previously attached.
detach checkpoint -- Detach from a checkpoint (experimental).
detach inferiors -- Detach from inferior ID (or list of IDS).
disconnect -- Disconnect from a target.
finish, fin -- Execute until selected stack frame returns.
...


(gdb) <b>help next</b>
next, n
Step program, proceeding through subroutine calls.
Usage: next [N]
Unlike "step", if the current source line calls a subroutine,
this command does not enter the subroutine, but instead steps over
the call, in effect treating it as a single source line.
</pre>

