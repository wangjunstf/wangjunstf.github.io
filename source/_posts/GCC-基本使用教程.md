---
title: GCC 基本使用教程
date: 2021-05-8 20:05:04
updated:
categories: [Linux]
tags: [gcc,g++,编译过程]
---

GCC全称GNU C Compiler，是以GPL协议发布的自由软件，其创始人理查德·马修·斯托曼是自由软件运动的精神领袖。

GNU 编译器套件包括 C、C++、Objective-C、Java、Ada 和 Go 语言前端，也包括了这些语言的库（如 libstdc++，libgcj等）

GCC 不仅支持 C 的许多“方言”，也可以区别不同的 C 语言标准；可以使用命令行选项来控制编译器在翻译源代码时应该遵循哪个 C 标准。例如，当使用命令行参数`-std=c99` 启动 GCC 时，编译器支持 C99 标准。
<!-- more -->
查看版本 gcc/g++ -v/--version

> g++相对gcc对代码书写规范更加严格，如果编译 c++ 语言,将其替换为 g++ 即可。

# 编译过程

高级语言源代码 -> 汇编语言 -> 机器语言

## 新建文件 t.c

`vim t.c`

```c++
#include <stdio.h>
int main(){
    printf("Hello world");
    return 0;
}
```

## 预处理

展开所有的include文件，包括头文件和宏定义，生成.i文件

`g++ -E t.c -o t.i`

## 编译

把目标代码编译为汇编代码，生成.s文件

`gcc -S t.cpp -o t.s`

## 汇编器

把指定源码编译为机器码.o文件包括（启动代码，目标代码，库代码，其它目标代码），但不进行链接

`gcc -c t.c -o t.o`

## 链接器

生成可执行 ELF 文件

gcc t.o -o t

# 常用参数

| gcc编译选项                             | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| -E                                      | 预处理指定的源文件，不进行编译                               |
| -S                                      | 编译指定的源文件，但是不进行汇编                             |
| -c                                      | 编译、汇编指定的源文件，但是不进行链接                       |
| -o [file1] [file2] / [file2] -o [file1] | 将文件 file2 编译成可执行文件 file1                          |
| -I directory                            | 指定 include 包含文件的搜索目录                              |
| -g                                      | 在编译的时候，生成调试信息，该程序可以被调试器调试           |
| -D                                      | 在程序编译的时候，指定一个宏                                 |
| -w                                      | 不生成任何警告信息                                           |
| -Wall                                   | 生成所有警告信息                                             |
| -On                                     | n的取值范围：0~3。编译器的优化选项的4个级别，-O0表示没有优化，-O1为缺省值，-O3优化级别最高 |
| -l                                      | 在程序编译的时候，指定使用的库                               |
| -L                                      | 指定编译的时候，搜索的库的路径。                             |
| -fPIC/fpic                              | 生成与位置无关的代码                                         |
| -shared                                 | 生成共享目标文件，通常用在建立共享库时                       |
| -std                                    | 指定C方言，如:-std=c99，gcc默认的方言是GNU C                 |

## 在程序中加入调试信息

新建源文件 t.c

```c
#include <stdio.h>

int main()
{

    int a = 10;

#ifdef DEBUG    
    printf("这是一条调试信息\n");
#endif

    int b, c, d, f;
    b = 10;
    c = b;
    d = c;
    f = d;
    for (int i = 0; i < 3; i++)
    {
        printf("hello gcc\n");
    }
    return 0;
}
```

使用两种方式可以输出 DEBUG，一种是在程序开头定义 DEBUG宏，例如 `#define DEBUG`

另一种是使用 gcc -D 命令选项。

`gcc t.c -o t -D DEBUG`      //DEBUG是在程序中指定的，可以指定多个

执行`./t`，就能输出 DEBUG信息

## 不生成任何警告

-w

gcc t.c -o t -w

## 生成所有的警告信息

-Wall

gcc t.c -o t -Wall

执行上述语句，则会输出：

t.c:4:9: warning: unused variable 'a' [-Wunused-variable]
    int a = 10;

## 编译器优化级别

-O0

-O1

-O2

-O3

级别越高，编译优化越大，越不容易通过反汇编得到原始代码。

## 包含调试信息

此时调试信息为源代码，该程序可被调试器调试，比如可被GDB调试

-g

gcc t.c -o t -D DEBUG -g //该程序还包括-D参数，调试时包括DEBUG信息

## 指定include 包含文件的搜索目录

gcc t.c -o t -I 路径     //I为i的大些

## 在程序编译时，指定使用的库

-l

gcc t.c -o t -l 库名称

## 在编译时，指定使用的库的路径

-L

gcc t.c -o t -L 库路径

## 生成与位置无关代码

-fpic/fPIC

gcc t.c -o t -fpic

## 生成共享目标文件，通常用在建立共享库时

-shared

gcc t.c -o t -shared

## 指定c方言

-std=c99

gcc t.c -o t -std=c99 以c99标准编译源文件
