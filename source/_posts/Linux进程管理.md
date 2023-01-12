---
title: Linux进程管理
date: 2021-04-16 15:41:02
updated:
categories: [Linux]
tags: [Linux,进程管理]
---
# CentOS 7的启动过程

1. BIOS加电自检
2. 进入Boot Loader
3. 加载Linux系统内核
4. 启动初始化程序（前期版本System V init，CentOS7中使用systems，采用并发启动机制，提升开启速度），该进程为系统的第一个进程。
5. systemd初始化系统，创建login进程。
6. 用户登录，login进程创建shell进程。以后进程都是由shell衍生出来。
<!-- more -->


systemd和System V init的区别与作用

| System V init运行级别 | systemd目标名称                     | 作用                               |
| --------------------- | ----------------------------------- | ---------------------------------- |
| 0                     | runlevel0.target，poweroff.target   | 关机                               |
| 1                     | runlevel1.target，rescue.target     | 单用户模式（救援模式）             |
| 2                     | runlevel2.target，multi-user.target | 等同于级别3                        |
| 3                     | runlevel3.target，multi-user.target | 多用户的文本界面（服务器缺省模式） |
| 4                     | runlevel4.target，multi-user.target | 等同于级别3                        |
| 5                     | runlevel5.target，graphical.target  | 多用户的图形界面                   |
| 6                     | runlevel5.target，reboot.target     | 重启                               |
| emergency             | emergency.target                    | 紧急Shell                          |

改变系统默认的运行目标方法：ln命令将目标文件链接到/etc/systemd/system/目录即可

如：ln -sf /lib/systemd/system/muti-user.target /etc/systemd/system/default.target



# Linux中的进程

根据进程PID区分不同的进程。

系统启动后的第一个进程是systemd，它的PID为1。

当系统启动以后，systemd进程会创建login进程等待用户登录系统。

当用户登录系统后，login进程就会为用户启动shell进程。

此后用户运行的进程都是由shell衍生出来的。

进程的另外4个识别号：实际用户识别号（real user ID），实际群组识别号（real group ID），以及有效用户识别号（effect user ID）和有效群组识别号（effect group ID）。



## 进程启动方式

手工启动

调度启动



## 进程的优先级

每个进程有一个默认优先级NICE值(0)

通过nice命令可以调整进程的NICE值

在CentOS中它的调整范围为-20~19，**NICE的值越大，进程的优先级越低**。



## 进程的状态

可执行态（runnable）

睡眠态（sleeping）

暂停态（stopped）

僵死态（zombie）

> 任何一个子进程(init除外)在exit()之后，并非马上就消失掉，而是留下一个称为僵尸进程(Zombie)的数据结构，等待父进程处理。

## Linux进程状态变换

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-16%20%E4%B8%8B%E5%8D%883.53.12.png" alt="截屏2021-04-16 下午3.53.12" style="zoom:50%;" />



# 作业

正在执行的一个或多个相关进程被称为作业。

一个作业可以包含一个或者多个进程。

作业可以分为两类：前台作业和后台作业

在某一时刻，每个用户只能有一个前台作业



# 进程操作相关命令

1. ps
2. jobs
3. pstree
4. top
5. nice和renice
6. kill

## ps命令

命令格式：ps [option]

| 常用选项 | 说明                             |
| -------- | -------------------------------- |
| -a       | 显示当前控制终端的所有进程       |
| -e       | 在命令后显示环境变量             |
| -u       | 显示进程的用户名和启动时间等信息 |
| -A       | 显示所有终端的进程               |
| -u       | 显示进程所有者的信息             |
| -x       | 显示没有控制终端的进程           |
| -f       | 显示进程树                       |
| -w       | 宽行输出，不截取输出中的命令行   |
| -l       | 按长格式显示输出                 |



## jobs命令

功能：查看系统当前的所有作业

命令格式：jobs [options]

常用选项：

-p：仅显示进程号

-l：同时显示进程号和作业号

-r：只列出运行的作业

-s：只列出停止的作业



## pstree

命令格式：pstree [option]

常用选项：

-a：显示每个程序的完整指令，包括路径，参数或是常驻服务的标示

-l：采用长列格式显示树状图

-p：显示进程的PID号

-u：显示用户名称

-V：显示版本信息

pid|user：根据pid或者user信息来显示我们需要的信息

例如：

```shell
pstree -p    #以树状图显示进程，进程号及进程ID
pstree -a    #以树状图显示所有进程的所有详细信息
pstree 1     #以树状图显示PID为1的进程以及子孙进程
pstree -p 1  #以树状图显示PID为1的进程以及子孙进程，同时显示每个进程的PID
```

## top

top命令是Linux下常用的性能分析工具，能够实时显示各个进程的资源占用情况。

命令格式：top [option]

常用选项:

-b 批处理

-c 显示完整的命令

-l 忽略失效过程

-s 保密模式

-S 累积模式

-i<时间> 设置间隔时间

-u<用户名> 指定用户名

-p<进程号> 指定进程

-n<次数>循环显示的次数

-d<秒> 信息更新时间

例如:

top -c //显示完整命令



**top的交互命令**

h 显示帮助画面，给出一些简短的命令总结说明

k 终止一个进程

i 忽略闲置和僵死进程。这是一个开关式命令。

q 退出程序

r 重新安排一个进程的优先级别

m 切换显示内存信息

t 切换显示进程和CPU状态信息

c 切换显示命令名称和完整命令行

M 工具驻留内存大小进行排序

P 根据CPU使用百分比大小进行排序

T 根据时间/累计时间进行排序



## nice 和renice命令

设置进程优先级的命令

1. nice命令

   功能：指定将要启动进程的优先级。不指定优先级时默认为0

   命令格式：nice [-n number] command [arg...]

   如: nice -n 10 ls

   ​	  nice -n 10 more 1.txt 

2. renice命令

   功能：修改**运行中的进程**的优先级。即设定指定用户或群组的进程的优先级

   命令格式：renice priority-number [-p PID] [-u user] [-g pgrps]

   主要选项说明：

   -p：进程号，指定指定进程的优先级

   -u：用户号，修改指定用户所启动进程的默认优先级

   -g：组群号，修改指定群组中所有用户所启动进程的默认优先级

   例如：

   ​		renice -15 -p 2423(PID)

## 终止进程命令（kill, killall, pkill）

功能：杀死进程，同时杀死该进程的所有子进程。

命令格式：kill [-signal] -PID

主要选项：

-l：信号，如果不加信号的编号参数，则使用"-l"参数会列出全部信号

-a：当处理当前进程时，不限制命令名和进程号的对应关系

-p：指定kill命令只打印相关进程的进程号，而不发送任何信号

-s：指定发送信号

-u：指定用户