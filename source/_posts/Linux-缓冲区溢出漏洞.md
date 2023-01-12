---
title: Linux 缓冲区溢出漏洞
date: 2021-09-17 13:50:09
updated:
categories: [信息安全]
tags: [Linux,缓冲区溢出,信息安全]
---
缓冲区溢出是一种非常普遍的漏洞，它的原理为输入大量的数据，超出了缓冲区的大小，且系统没有对输入数据的长度进行检查，这样数据可能覆盖了内存的重要区域，例如返回地址等，攻击者只需制作特定的数据，就可以让受攻击的计算机执行指定的代码。为了重现一些经典的缓冲区溢出漏洞，需要关闭 Linux 下针对这方面的一些保护措施，例如：

##  SSP( Stack Smashing Protector )
<!-- more -->
SSP 是 gcc 提供的针对栈上缓冲区溢出提供的检查机制。典型的缓冲区溢出攻击会构造输入数据来覆盖缓冲区之外的数据，尤其是函数返回地址。启用 SSP机制后，在编译器生成的代码中，对于那些存在缓冲区的函数，函数开始时会在对应栈帧中压入一个随机值，当函数快要返回时，检查该随机值是否发生了变化，如果发生了变化，就将控制流转移到特定的函数。比较常见为输出以下信息，并退出进程。

```shell
*** stack smashing detected ***: <unknown> terminated
Aborted (core dumped)
```

使用以下代码进行说明：

```c
#include <stdio.h>
#include <string.h>

void hacked(){
    printf("I am hacked");
}

void foo(char *s){
    char buffer[20];
    strcpy(buffer,s);
    printf("You entered: %s\n",buffer);
}

int main(){
    char buff[300];
    printf("Enter some text: ");
    scanf("%s",buff);
    foo(buff);
    return 0;
}
```



使用 GDB 查看 foo 的反汇编代码：

```assembly
(gdb) info function
All defined functions:

Non-debugging symbols:
0x000003e0  _init
0x00000420  printf@plt
0x00000430  strcpy@plt
0x00000440  __libc_start_main@plt
0x00000450  __isoc99_scanf@plt
0x00000460  __cxa_finalize@plt
0x00000468  __gmon_start__@plt
0x00000470  _start
0x000004b0  __x86.get_pc_thunk.bx
0x000004c0  deregister_tm_clones
0x00000500  register_tm_clones
0x00000550  __do_global_dtors_aux
0x000005a0  frame_dummy
0x000005a5  __x86.get_pc_thunk.dx
0x000005a9  hacked
0x000005d4  foo
0x00000614  main
0x00000680  __x86.get_pc_thunk.ax
0x00000690  __libc_csu_init
0x000006f0  __libc_csu_fini
0x000006f4  _fini
```

由上述代码可知，在函数开始，gs:0x14 的值被存储在 ebp - 0xc，在函数返回之前对 ebp - 0xc处的值进行检查，如果和 gs:0x14 不一样，说明发生了溢出，紧接着执行__stack_chk_fail_local 并退出进程。

根据汇编代码，可以知道foo 函数中 buffer 缓冲区的位置为 ebp-0x20。当我们输入的数据长度大于缓冲区的大小时，就可能覆盖缓冲区以外的内容。在函数结束时，就可能被检查出来。

```shell
$ gcc -g -m32 t.c -o t
mygit@ubuntu:~/webServer-master/test$ ./t
Enter some text: cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
You entered: cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
*** stack smashing detected ***: <unknown> terminated
Aborted (core dumped)
```

在 gcc 中  SSP 是默认开启的。使用以下命令开启和关闭 SSP。日常建议开启，如果需要对简单的缓冲区漏洞进行重现，就需要关闭该选项。

```shell
-fstack-protector    //编译时启用SSP机制
-fno-stack-protector //编译时禁用SSP机制
```



## DEP( Data Execution Prevention )

某一类缓冲区溢出攻击通过将攻击指令存储在栈上，然后通过缓冲区溢出，修改函数的返回地址为栈上攻击指令的首地址。这样当函数返回时，函数的执行流就跳转到栈上构造好的指令序列。在现代编译器中，为防止栈中的数据被作为指令执行，使用了 DEP机制，即限制内存的属性。使得栈(可写不可执行)，代码段(可执行不可写)。

使用 gcc 编译时，使用 -z execstack 参数来让最终的可执行程序中的栈具有可执行权限。

```
-z execstack    //设置栈内存段具备可执行权限
```

在程序运行时，可通过cat /proc/pid/maps 查看 pid 所对应的进程的内存映射情况,其中包括对进程的段的属性描述。例如：

```
$ cat /proc/28214/maps
565ee000-565ef000 r-xp 00000000 08:01 1074931                            /home/mygit/webServer-master/test/t
565ef000-565f0000 r--p 00000000 08:01 1074931                            /home/mygit/webServer-master/test/t
565f0000-565f1000 rw-p 00001000 08:01 1074931                            /home/mygit/webServer-master/test/t
57c81000-57ca3000 rw-p 00000000 00:00 0                                  [heap]
f7ce0000-f7eb2000 r-xp 00000000 08:01 1966086                            /lib32/libc-2.27.so
f7eb2000-f7eb3000 ---p 001d2000 08:01 1966086                            /lib32/libc-2.27.so
f7eb3000-f7eb5000 r--p 001d2000 08:01 1966086                            /lib32/libc-2.27.so
f7eb5000-f7eb6000 rw-p 001d4000 08:01 1966086                            /lib32/libc-2.27.so
f7eb6000-f7eb9000 rw-p 00000000 00:00 0 
f7ed9000-f7edb000 rw-p 00000000 00:00 0 
f7edb000-f7ede000 r--p 00000000 00:00 0                                  [vvar]
f7ede000-f7edf000 r-xp 00000000 00:00 0                                  [vdso]
f7edf000-f7f05000 r-xp 00000000 08:01 1966082                            /lib32/ld-2.27.so
f7f05000-f7f06000 r--p 00025000 08:01 1966082                            /lib32/ld-2.27.so
f7f06000-f7f07000 rw-p 00026000 08:01 1966082                            /lib32/ld-2.27.so
ffabc000-ffadd000 rw-p 00000000 00:00 0                                  [stack]
```



## ASLR( address space layout randomization )

在基本的缓冲区溢出攻击中，最基本的步骤就是定位某些目标的内存映射地址。例如最简单的 shellcode 注入需要在栈上构造特定指令序列，并通过缓冲区溢出使用攻击指令序列地址覆盖函数的返回地址。ret2libc 方法则需要定位内存中的标准库函数的内存映射地址。ASLR ，即内存布局随机化机制由操作系统实现，主要被划分为映像随机化、栈随机化和堆随机化这几类，分别针对程序的加载基地址、栈基址和堆基址进行随机化。



编译好的程序，它的各个段的加载地址是固定的，也就是程序运行时它的各个段的地址是固定的。通过查看 ELF 文件的 Program header table 信息，可以知道需加载入内存的各个段的基地址。通过命令 `objdump -h`查看，例如：

```shell
$ objdump -h t

t:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .interp       00000013  00000154  00000154  00000154  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.ABI-tag 00000020  00000168  00000168  00000168  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .note.gnu.build-id 00000024  00000188  00000188  00000188  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .gnu.hash     00000020  000001ac  000001ac  000001ac  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
..
  8 .rel.dyn      00000040  000003bc  000003bc  000003bc  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  9 .rel.plt      00000028  000003fc  000003fc  000003fc  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 10 .init         00000023  00000424  00000424  00000424  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 11 .plt          00000060  00000450  00000450  00000450  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 12 .plt.got      00000010  000004b0  000004b0  000004b0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 13 .text         000002e4  000004c0  000004c0  000004c0  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 14 .fini         00000014  000007a4  000007a4  000007a4  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 15 .rodata       0000003a  000007b8  000007b8  000007b8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 ...
 20 .dynamic      000000f8  00001ed4  00001ed4  00000ed4  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 21 .got          00000034  00001fcc  00001fcc  00000fcc  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 22 .data         00000008  00002000  00002000  00001000  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 23 .bss          00000004  00002008  00002008  00001008  2**0
                  ALLOC
 24 .comment      00000029  00000000  00000000  00001008  2**0
                  CONTENTS, READONLY
```

对于运行中的程序，使用 cat /proc/pid/maps，查看 pid 的内存映射情况，例如：

```
$ cat /proc/28214/maps
565ee000-565ef000 r-xp 00000000 08:01 1074931                            /home/mygit/webServer-master/test/t (deleted)
565ef000-565f0000 r--p 00000000 08:01 1074931                            /home/mygit/webServer-master/test/t (deleted)
565f0000-565f1000 rw-p 00001000 08:01 1074931                            /home/mygit/webServer-master/test/t (deleted)
57c81000-57ca3000 rw-p 00000000 00:00 0                                  [heap]
f7ce0000-f7eb2000 r-xp 00000000 08:01 1966086                            /lib32/libc-2.27.so
f7eb2000-f7eb3000 ---p 001d2000 08:01 1966086                            /lib32/libc-2.27.so
f7eb3000-f7eb5000 r--p 001d2000 08:01 1966086                            /lib32/libc-2.27.so
f7eb5000-f7eb6000 rw-p 001d4000 08:01 1966086                            /lib32/libc-2.27.so
f7eb6000-f7eb9000 rw-p 00000000 00:00 0 
f7ed9000-f7edb000 rw-p 00000000 00:00 0 
f7edb000-f7ede000 r--p 00000000 00:00 0                                  [vvar]
f7ede000-f7edf000 r-xp 00000000 00:00 0                                  [vdso]
f7edf000-f7f05000 r-xp 00000000 08:01 1966082                            /lib32/ld-2.27.so
f7f05000-f7f06000 r--p 00025000 08:01 1966082                            /lib32/ld-2.27.so
f7f06000-f7f07000 rw-p 00026000 08:01 1966082                            /lib32/ld-2.27.so
ffabc000-ffadd000 rw-p 00000000 00:00 0                                  [stack]
```

在过去的系统环境中，程序的.text、.bss、.rodata等段地址在加载入内存时是确定的，程序运行时进程中 stack 和 heap 的起始地址也总是固定的。这样攻击者就更容易定位到特定代码的地址。ALSR 机制后，操作系统会在加载段时在其原始的基地址上加上一个随机值，这样同一程序多次运行，它的内存布局都会不一样，这样攻击者更难实施特定的攻击。

使用命令 `cat /proc/sys/kernel/randomize_va_space`查看 ASLR 的设置情况：

* 0:  ASLR未启用
* 1：ALSR 机制会随机化 stack、vdso和 mmap 的起始基地址
* 2：对上述目标进行随机化外还会对堆基地址进行随机化。

通过命令 `echo 0 > /proc/sys/kernel/randomize_va_space` 关闭 ALSR，或设置为其它值。

上述命令为全局生效，且需要 root 权限，使用以下命令只在当前终端中关闭 ALSR。

```
　setarch `uname -m` -R /bin/bash
```



## 通过 GDB 获得目标地址

使用 GDB 可以确定目标代码的地址，例如目标缓冲区的地址。

使用以下代码查看 foo 的汇编代码：



```
   disassemble foo
   Dump of assembler code for function foo:
   0x000005d4 <+0>:     push   %ebp
   0x000005d5 <+1>:     mov    %esp,%ebp
   0x000005d7 <+3>:     push   %ebx
   0x000005d8 <+4>:     sub    $0x24,%esp
   0x000005db <+7>:     call   0x66b <__x86.get_pc_thunk.ax>
   0x000005e0 <+12>:    add    $0x19f0,%eax
   0x000005e5 <+17>:    sub    $0x8,%esp
   0x000005e8 <+20>:    push   0x8(%ebp)
   0x000005eb <+23>:    lea    -0x1c(%ebp),%edx
   0x000005ee <+26>:    push   %edx
   0x000005ef <+27>:    mov    %eax,%ebx
   0x000005f1 <+29>:    call   0x430 <strcpy@plt>
   0x000005f6 <+34>:    add    $0x10,%esp
   0x000005f9 <+37>:    nop
   0x000005fa <+38>:    mov    -0x4(%ebp),%ebx
   0x000005fd <+41>:    leave  
   0x000005fe <+42>:    ret    
```

由上述代码可知，foo 函数内 buffer 缓冲区的地址为：ebp - 0x1c

```
(gdb) info function
All defined functions:

Non-debugging symbols:
0x000003e0  _init
0x00000420  printf@plt
0x00000430  strcpy@plt
0x00000440  __libc_start_main@plt
0x00000450  __isoc99_scanf@plt
0x00000460  __cxa_finalize@plt
0x00000468  __gmon_start__@plt
0x00000470  _start
0x000004b0  __x86.get_pc_thunk.bx
0x000004c0  deregister_tm_clones
0x00000500  register_tm_clones
0x00000550  __do_global_dtors_aux
0x000005a0  frame_dummy
0x000005a5  __x86.get_pc_thunk.dx
0x000005a9  hacked
0x000005d4  foo
0x00000614  main
0x00000680  __x86.get_pc_thunk.ax
0x00000690  __libc_csu_init
0x000006f0  __libc_csu_fini
0x000006f4  _fini
```

由上述信息可知，foo 的地址 和  hacked 的地址。



## 总结

在 GDB 中看到的地址不一定是真实程序运行的地址，原因多种多样，例如环境变量不一致会导致地址差异。Linux 近年来为了预防缓冲区漏洞的产生，也采取了多种措施，不只是上述提到的三种。在增强了 Linux 安全性的同时，也大大提高了缓冲区溢出攻击的门槛。