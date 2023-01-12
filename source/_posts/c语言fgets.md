---
title: c语言 fgets()函数
date: 2021-04-8 00:36:10
updated: 
categories: [C]
tags: [c,fgets]
---
fgets函数用于从指定流读取指定字节数的字符，Ascii码字符1个字符占用1个字节，1个汉字占用多少字节因平台而异。

**验证一个汉字占用几个字节：**

```c
char chinese[10] = "中文";
printf("%ld\n", strlen(chinese));
```
<!-- more -->
终端输出：6，   表示当前平台一个汉字占用3个字节。

# 函数调用方法

char*  fgets(char* str, int num, FILE* stream );

## 函数形参

* str为目标数组指针

* num为读取的字符数，包括**换行符**，意味着只能输入(num-1)个Ascii码字符。

  > 输入为英文字符：
  >
  > 实际能输入(num-1)个字符，最后一个字符保存字符串结束符，即'\0'
  >
  > 输入为汉字:
  >
  > 实际能输入(num-1)/3个字符，最后一个字符保存字符串结束符，即'\0'
  >
  > 混合汉字与英文的输入：
  >
  > 需满足 (m + n*3 + 1) ⋜ num，其中m为英文字符数，n为中文字符数
  >
  > num⋜ sizeof(str)

* stream为输入流

  > 指向作为输入流的FILE对象指针
  >
  > 传递stdin，表示从标准输入(终端)读取数据



## 函数返回值

根据执行结果有以下几种返回情况：

* 执行成功：返回str

* 执行失败：返回空指针

  > 设置错误指示符：ferror
  >
  > 例如在调用fgets之后，紧接着执行int ferror（FILE * stream）;
  >
  > * 返回非0值，则表示之前对stream流操作失败
  >
  > * 返回0，表示之前对stream流操作成功

* 其他情况

  **读取过程中，遇到文件结束符号，则设置eof指示符（[feof](https://www.cplusplus.com/feof)）并返回str**

  **未读取到任何字符前遇到文件描述符，返回空指针**

  

# 代码示例

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define BUF_SIZE 1024
int itoc(int num, char* str)
{
    char tem[1024];
    int id=0,id2=0;
    while (num)
    {
        int t = num % 10;
        tem[id++] = t + '0';
        num /= 10;
    }
    str[id--] = '\0';
    while (id >= 0)
    {
        str[id2++] = tem[id--];
    }
    return 0;
}

int main(int argc, char *argv[])
{
    char str[BUF_SIZE];

    fgets(str,BUF_SIZE, stdin);
    int len = strlen(str);
    printf("str'length %d\n", len);
    printf("输入的字符串：%s",str);
    printf("下标为:%d 字符为%c 字符Ascii为%d\n下标为:%d 字符为%c 字符Ascii为%d\n", len-1, str[len-1], (int)str[len-1], len, str[len], (int)str[len]);
    return 0;
}
```

编译并运行上述代码：输入hello world

输出以下内容

```shell
str'length 12
输入的字符串：hello world         #   实际读到的字符串为"hello world\n"
下标为:11 字符为                  #   str[11]为换行符
 字符Ascii为10                   #   str[12]为行结束符
下标为:12 字符为 字符Ascii为0
```

注意：实际输入的字节数，不要超过BUF_SIZE-2，因为还要额外预留两个空间存储换行符和行结束符