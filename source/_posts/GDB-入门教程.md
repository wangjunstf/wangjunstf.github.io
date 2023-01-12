---
title: GDB 入门教程
date: 2021-06-12 23:18:37
updated:
categories: [Linux]
tags: [gdb,调试]
---
假设当前目录下共有以下3个文件，接下来会利用以下代码来演示GDB的调试过程。

├── main.cpp

├── tool.h

└── tools.cpp

文件内容分别为：

tools.h：函数声明

```cpp
#ifndef _TOOLS_H_
#define _TOOLS_H_
void greeting();
int sum(int a, int b);

#endif /* tools.h */
```



tools.cpp：函数定义

```cpp
#include <iostream>
#include "tools.h"

using namespace std;

void greeting()
{
    cout << "Hello World" << endl;
}

int sum(int a, int b)
{
    return a + b;
}
```



main.cpp：主函数

```cpp
#include <iostream>
#include <string.h>
#include "tool.h"
using namespace std;

void greeting();
int main(int argc, char* argv[]){
    int s = 0;
    int m=0;
    int n=0;
    for(int i=1; i<=atoi(argv[1]); i++){
        s+=i;
        m++;
        n++;
    }
    printf("sum=%d\n",s);
    greeting();
    cout<<sum(12,12)<<endl;
    print_num();
    return 0;
}
```

以上代码只是为了演示之用，并无特别之处。

编译方式

```shell
// -g 添加调试信息    -Wall 输出所有警告信息，例如定义了从未使用过的变量
$ gcc main.cpp -o main -g -Wall  
```



## 启动

调试可执行程序

```shell
$gdb main
```



调试 core 文件，core 文件是程序运行过程中出现Segmentation fault (core dumped)错误时，程序停止运行时产生的。core文件是程序运行状态的内存映象。使用gdb调试core文件，可以帮助我们快速定位程序出现段错误的位置。可执行程序编译时应加上-g编译选项，生成调试信息。

```shell
$gdb <program> <core dump file>
```



调试服务程序

```
$gdb <program> <PID>
```



## 设置和获取参数

有时需要在执行程序时输入额外的参数，例如：像下面这样执行程序test

```shell
$ ./main 5
```

调试时，往往直接执行`$ gdb main`

那么怎么在gdb中输入该程序的参数呢？

```shell
(gdb) set args 5
```

还可以查看程序的参数

```shell
(gdb) show args
Argument list to give program being debugged when it is started is "5".
```



## 查看代码

GDB提供了两种查看源码的方式，分别是**根据行号**和**函数名**查看。除了可以查看本文件源码，还可以查看其他文件的源码。

### 本文件

本文件指的是该程序对应的main函数所在文件，即main.cpp

```shell
(gdb) l        # 每执行1次显示10行，再执行1次显示次10行
(gdb) l 15     # 显示第15行，此时会将15行显示在屏幕窗口中央，方便查看前后的代码
(gdb) l main   # 显示本文件的main函数
```



### 其他文件

共同编译的所有文件中，除了main函数所在文件的其它文件。在这里，除了main.cpp即tools.cpp

```shell
(gdb) l tools.cpp:15       		# 在tools.cpp中，显示第15行附近的代码
(gdb) l tools.cpp:sum      		# 在tools.cpp中，查看sum函数的代码
```



### 设置和获取显示行数

```shell
(gdb) show list    				# 显示行数
(gdb) set list 20  				# 设置行数
```

## 断点

可以在一次调试中设置1个或多个断点，下一次只需让程序自动运行到设置断点位置，便可在上次设置断点的位置中断下来，极大的方便了操作，同时节省了时间。

可以根据行号，函数名设置断点，还可根据条件设置断点(一般用于循环中)

### 本文件

```shell
(gdb) b 10              # 将第10行设置为断点
(gdb) b main            # 将main函数入口处设置为断点
(gdb) l                 # 可以看到在main.cpp中含有greeting函数的声明
11
12	void greeting();
13	int main(int argc, char* argv[]){
14	    int s = 0;
15	    int m=0;
16	    int n=0;
17	    for(int i=1; i<=atoi(argv[1]); i++){
18	        s+=i;
19	        m++;
20	        n++;
(gdb) b greeting        # 此时设置的断点并不是函数的声明处，而是函数的定义处，greeting函数定义在tools.cpp文件中
Breakpoint 1 at 0xaa2: file tools.cpp, line 13.
```



### 其他文件

```shell
b tools.cpp:12            # 将tools.cpp的第12行设置为断点
b tools.cpp:sum           # 将tools.cpp的sum函数设置为断点
```



### 设置条件断点

条件断点一般用于循环中

本文件

```shell
# 设置i==2时，第18行为断点
# 行号必须在变量的作用域范围内
(gdb) b 18 if i==2                              
Breakpoint 1 at 0x9d8: file main.cpp, line 18.
```



其他文件

```shell
# 设置tools.cpp文件内，i==5时，第22行为断点
# 22必须在i的作用域范围
(gdb) b tools.cpp:22 if i==5                  
Breakpoint 2 at 0xafb: file tools.cpp, line 22.
```



### 查看和删除断点

```shell
(gdb) i b            # 显示所有断点
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000000009a4 in main(int, char**) at main.cpp:14
2       breakpoint     keep y   0x00000000000009ec in main(int, char**) at main.cpp:22
3       breakpoint     keep y   0x0000000000000a39 in main(int, char**) at main.cpp:25

d 1                  # 删除第1个断点                               
```



### 设置断点无效和有效

```shell
clear n							# 清除第n行的断点
dis 2               # 将第2个断点设置为无效
ena 2               # 将第2个断点设置有效 
delete breakpoints  # 删除所有断点
```



## 运行

有两种运行方式，一种是从主函数开始运行，一种是运行到第1个断点处

### 从主函数运行

程序从main函数开始

```
(gdb) run
```



### 运行到第1个断点处

程序停在第一个断点处

```
(gdb) start
```



### 执行流控制

```shell
c/continue         # 向下运行到下一个断点处
n/next             # 执行下一行代码，不进入调用的函数，直接返回结果
s/step             # 执行下一行代码，进入调用的函数
finish             # 跳出函数体
until    					 # 跳出当前循环 在执行完循环体内的最后一条语句之后执行 until，才可以跳出循环
until+行号          # 运行至某行，不仅仅用来跳出循环
call 函数名称(参数)      # 调用程序中可见的函数，并传递参数，如：call gdb_test(67)
```



### 打印变量

```shell
p  变量名                      # 打印变量值，可以答应在当前作用域之内的变量名
ptype 变量名                   # 打印变量类型
```



## 查询

### 自动变量操作

可以在每次执行时都打印该变量的值，常用于循环体中

```shell
display 变量名                 # 自动打印指定变量的值
i display         						# 查看所有设置自动打印的变量值
undisplay 编号     						 # 取消自动打印变量值
print 表达式                   # 打印表达式的值，可以是任何有效表达式
print num                     # 打印整数num
print ++num                   # 打印num+1 后的值
print gdb_test(1024)          # 以整数 22 作为参数调用 gdb_test()函数
print gdb_test(num)           # 以变量 num 作为参数调用 gdb_test() 函数
watch 表达式 									# 设置一个监视点，一旦被监视的“表达式”的值改变，gdb将强行终止正在被调试的程序。如： watch num
whatis 变量名                  # 显示某个变量的类型
info function		              # 查询函数信息
info locals										# 显示当前堆栈页的所有变量

```



### 运行时信息

```shell
where/bt 						# 当前运行的堆栈列表；
bt backtrace        # 显示当前调用堆栈
up/down             # 改变堆栈显示的深度
info program        # 查看程序的是否在运行，进程号，被暂停的原因。
```



### 设置变量值

```shell
set var 变量名=变量值           # 可以临时改变该变量的值               
```

## 多窗口调试

使用 layout 可以同时在多个窗口调试代码

```shell
layout src        # 显示源代码窗口
layout asm        # 显示反汇编窗口
layout regs       # 显示源代码/反汇编和CPU寄存器窗口
layout split      # 显示源代码和反汇编窗口
Ctrl + L					# 刷新窗口
```



## 多进程调试

GDB 默认调试的是父进程，使用以下命令切换调试的进程

```shell
set follow-fork-mode child      # 调试子进程
set follow-fork-mode parent     # 调试父进程
show follow-fork-mode           # 显示设置信息
```

GDB 可以同时调试一个进程，也可以同时调试多个进程，使用以下命令设置：

```shell
set detach-on-fork <mode>       # 当 mode 为 on 时，表示程序只调试一个进程（可以是父进程、子进程）。当 mode 为 off 时，父子进程都在gdb的控制之下，其中一个进程正常的调试，另一个会被设置为暂停状态。
```

GDB 将每一个被调试程序的执行状态记录在一个名为 inferior 的结构中。一般情况下一个 inferior 对应一个进程，不同的 inferior 有不同的地址空间。inferior 有时候会在进程没有启动的时候就存在。

```shell
info inferiors                 # 显示所有的 inferior,当前调试的进程前有 "*"。
```

 切换进程使用命令 inferior

```shell
inferior <num>                 # 切换到 num 号进程
```

设置捕获点中断，调用 fork 函数时将产生中断

```shell
catch fork
```



## 查看内存

命令格式：例如 `x/<n/f/u> <addr>`

例如： 

```shell
(gdb) x/3cb 0x8049098
0x8049098 <Snippet>:	75 'K'	65 'A'	78 'N'	
```

x(examine)为命令的简称，用于检查内存数据。

命令的意思是：查看从内存地址 0x8049098 开始的数据，每字节为单位(b)，以ASCII字母形式显示(c)，显示 3 个单位的数据(3)。



下面对 3cb 三个字符对应的含义进行讲解：

1. 3

   表示显示的单位数据个数，可根据需要自定义，和第三个字符参数有关。

2. c

   用什么格式来解析这些二进制位，因为内存本质上是二进制位，它可被解释为多种含义，例如：0111 0101，可表示十进制数 117，也可以表示十六进制数 75，也可以表示字符 K(ASCII 字母)

   可用的格式字符如下：

   c 按字符格式显示变量。

   x 按十六进制格式显示变量。

   d 按十进制格式显示变量。

   u 按十进制格式显示无符号整型。

   o 按八进制格式显示变量。

   t 按二进制格式显示变量。

   a 按十六进制格式显示地址，并显示距离前继符号的偏移量(offset)。常用于定位未知地址(变量)。

   f 按浮点数格式显示变量。

3. b

   表示从当前地址往后请求的位宽大小。如果不指定的话，GDB默认是4个bytes。也可以指定以下字符

   **b**表示单字节，**h**表示双字节，**w**表示四字 节，**g**表示八字节。



## 其它工具

### cgdb

cgdb可以看作gdb的界面增强版,用来替代gdb的 gdb -tui。cgdb主要功能是在调试时进行代码的同步显示。

