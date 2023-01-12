---
title: Linux umask chown chgrp
date: 2021-08-25 17:05:08
updated: 
categories: [Linux]
tags: [Linux,umask,chown,chgrp]
---
## umask命令

### 说明

umask命令指定在建立文件时预设的权限掩码。由3位八进制数字组成。
查看当前系统的文件掩码：`umask -S`
<!-- more -->
```shell
mygit@ubuntu:~/code$ umask
0002  #第一位保留
mygit@ubuntu:~/code$ umask -S
u=rwx,g=rwx,o=rx
mygit@ubuntu:~/code$ 
```

> Ubuntu 18.04.5 LTS 的默认掩码为：
>
> u=rwx,g=rwx,o=rx

### umask命令使用方式

umask [-S] maskcode

-S：用字符显示权限掩码

默认文件权限：666-umask

默认目录权限：777-umask

> 666为普通文件最高权限，777为目录最高权限

这里我有个疑问，为什么普通文件最高权限是666，目录最高权限是777？

通过查阅资料，总算知道了其中的奥秘。

> **读权限**
>
> 对文件：可以读取文件内容    对目录：可以浏览目录。
>
> **写权限**
>
> 对文件：可以新增，修改，删除文件内容    对目录：可以新建，删除，修改，移动目录内文件。
>
> **执行**
>
> 对文件：可以执行    对目录：可以进入该目录。
>
> 目录最高权限777，表示默认创建的目录，其所属用户和所属组除了读写外，还有进入目录的权限。
>
> 普通文件最高权限666，除了编译程序生成的可执行文件，通过其它方式创建的普通文件其所属用户和所属组默认只具有读和写权限，而没有执行权限。==通常情况下普通文件并不需要可执行权限，只有在必要时，可以给.sh等脚本文件赋予可执行权限==。
>
> 通常只有编译程序可以创建默认就可执行的文件，例如gcc。



应为系统默认umask为 0002，第1位为保留位

因此，默认创建的普通文件权限为664，即u=rw-，g=rw-，o=r--； 默认创建的目录权限为：775，即u=rwx，g=rwx，o=r-x

> ==默认创建的普通文件指的是由open(), creat(), mkdir(), mkfifo()函数创建的文件==。



## chown命令

### 说明

chown(change own)命令用于更改文件的所属用户和所属组

### 使用方法

```shell
chown mygit:mac t.txt   #将文件t.txt的拥有者改为mygit，所属组改为mac
chown mygit t.txt #将文件t.txt的拥有者改为mygit,所属组不变
chown :mac t.txt   #将文件t.txt的所属组改为mac，拥有者不变
```



### chown命令参数

| 参数 | 含义                     |
| ---- | ------------------------ |
| -R   | 递归改变所有子目录和文件 |
| -v   | 显示更改信息             |

例如

```shell
chown -R -v root:root t   #递归改变目录t及其子目录内所有文件的所属用户和所属组，并显示更改信息
```



## chgrp命令

### 说明

chgrp(change group)命令用于改变文件所属组

### 用法

```shell
chgrp root t.txt. #将t.txt的所属组改为bin
```


