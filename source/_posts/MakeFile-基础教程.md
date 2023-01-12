---
title: MakeFile 基础教程
date: 2021-09-10 10:26:53
updated:
categories: [Linux]
tags: [makefile,编译]
---

# Makefile 作用
Makefile的作用为实现自动化编译，主要为了解决以下问题：
- 有很多源文件需要编译时
- 当存在很多源文件时，只修改了个别源文件，这时只需要编译修改过的文件即可，而无需整个项目都编译一遍。
- 当多个源文件存在依赖关系时，需要先编译一些文件，后编译一些文件
# Makefile安装
<!-- more -->
要使用MakeFile，首先要按照make工具

在ubuntu18.0.4系统下，安装命令为：

sudo apt install build-essential

该命令会同时安装gcc/g++/make等工具



# Makefile文件命名和规则

## 文件命名

makefile或Makefile

## Makefile 规则

一个 Makefile 文件中可以有一个或者多个规则

```makefile
目标 ... : 依赖 ...
	命令（Shell 命令)
	...
```

目标：最终要生成的文件（伪目标除外）

依赖：生成目标所需要的文件或是目标

命令：通过执行命令对依赖操作生成目标（命令前必须 Tab 缩进）

Makefile 中的其它规则一般都是为第一条规则服务的。

## 基本原理

1.命令在执行之前，需要先检查规则中的依赖是否存在

​	a.如果存在，执行命令

​	b.如果不存在，向下检查其它的规则，检查有没有一个规则是用来生成这个依赖的，如果找到了，则执行该规则中的命令

2.检测更新，在执行规则中的命令时，会比较目标和依赖文件的时间

​	a.如果依赖的时间比目标的时间晚，需要重新生成目标

​	b.如果依赖的时间比目标的时间早，目标不需要更新，对应规则中的命令不需要被执行

## 变量

### 自定义变量

变量名=变量值  

```makefile
var=hello
#获取变量的值 $(变量名)
$(var)
```

### 预定义变量

AR : 归档维护程序的名称，默认值为 ar

CC : C 编译器的名称，默认值为 cc

CXX : C++ 编译器的名称，默认值为 g++

$@ : 目标的完整名称

$< : 第一个依赖文件的名称

$^ : 所有的依赖文件

```makefile
app:main.c a.c b.c
	gcc -c main.c a.c b.c -o app
	
#自动变量只能在规则的命令中使用
app:main.c a.c b.c
	$(CC) -c $^ -o $@
```

## 模式匹配

```makefile
add.o:add.c
      gcc -c add.c
div.o:div.c
      gcc -c div.c
sub.o:sub.c
      gcc -c sub.c
mult.o:mult.c
      gcc -c mult.c
main.o:main.c
      gcc -c main.c
```

%.o:%.c

​	%: 通配符，匹配一个字符串

​	两个%匹配的是同一个字符串

%.o:%.c

​	gcc -c $< -o $@



## 函数

**$(wildcard PATTERN...)**

- 功能：获取指定目录下指定类型的文件列表

- 参数：PATTERN 指的是某个或多个目录下的对应的某种类型的文件，如果有多个目录，一般使用空格间隔

- 返回：得到的若干个文件的文件列表，文件名之间使用空格间隔

- 示例：获取 当前目录下和sub目录中所有的 c 源文件。

  ​	$(wildcard \*.c ./sub/\*.c)    

  ​	返回值格式: a.c b.c c.c d.c e.c f.c



$(patsubst \<pattern\>,\<replacement\>,\<text\>)

- 功能：查找\<text\>中的单词(单词以“空格”、“Tab”或“回车”“换行”分隔)是否符合模式\<pattern\>，如果匹配的话，则以\<replacement\>替换。

- \<pattern\>可以包括通配符`%`，表示任意长度的字串。如果\<replacement\>中也包含`%`，那么，\<replacement\>中的这个`%`将是\<pattern\>中的那个%所代表的字串。(可以用`\`来转义，以`\%`来表示真实含义的 % 字符)

- 返回：函数返回被替换过后的字符串列表，不更改任何文件

- 示例：将当前目录前所有的 .c 文件都替换为 .o 文件返回，

  ​	$(patsubst %.c, %.o, x.c bar.c)

  ​	返回值格式: x.o bar.o



# Makefile实战

## 源文件

├── add.c
├── main.c
├── math.h
└── subtract.c



文件内容分别为：

main.c

```c
#include <stdio.h>

#include "math.h"


int main(){

    int a = 12;

    int b = 2;

    printf("a=%d b=%d\n",a,b);

    printf("a+b=%d\n",add(a,b));

    printf("a-b=%d\n",subtract(a,b));

    return 0;

}
```



自定义头文件

math.h

```c
#ifndef math_h

#define math_h

int add(int a,int b);

int subtract(int a,int b);

#endif
```



头文件里两个函数的实现：

add.c

```c
#include "math.h"

int add(int a,int b){
    return a+b;
}
```



subtract.c

```c
#include "math.h"

int subtract(int a,int b){
    return a-b;
}
```



## 不用Makefile

如果要生成可执行文件，需要编译main.c add.c subtract.c

编译命令为：

gcc main.c add.c subtract.c -o app



## 用Makefile

### 第一版Makefile

```makefile
app:main.c add.c subtract.c
	gcc main.c add.c subtract.c -o app
```



### 第二版Makefile

```makefile
app:main.o add.o subtract.o

    gcc main.o add.o subtract.o -o app

main.o:main.c

    gcc -c main.c -o main.o

add.o:add.c

    gcc -c add.c -o add.o

subtract.o:subtract.c

    gcc -c subtract.c -o subtract.o
```



### 第三版Makefile

\#定义变量

```makefile
src=subtract.o add.o main.o

target=app

$(target):$(src)

    $(CC) $(src) -o $(target)

add.o:add.c

    gcc -c add.c -o add.o

subtract.o:subtract.c

    gcc -c subtract.c -o subtract.o

main.o:main.c

    gcc -c main.c -o main.o
```

### 第四版Makefile

```makefile
#定义变量

src=subtract.o add.o main.o

target=app


$(target):$(src)

    $(CC) $(src) -o $(target)


%.o:%.c

    $(CC) -c $< -o $@
```

### 第五版Makefile

```makefile
src=$(wildcard ./*.c)

objs=$(patsubst %.c, %.o, $(src))

target=app


# 将所有的.o 文件编译为可执行程序 
$(target):$(objs)
    $(CC) $(objs) -o $(target)


# $< 表示第一个依赖项,$@ 表示目标，将所有的源文件编译为 .o 文件(目标文件，不进行链接)
%.o:%.c
    $(CC) -c $< -o $@


#clean为伪目标

.PHONY:clean

clean:
    rm $(objs) -f

```