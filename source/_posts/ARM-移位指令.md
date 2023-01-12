---
title: ARM 移位指令
date: 2022-01-20 11:36:23
updated:
categories: [ARM]
tags: [ARM, 移位指令]
---
移位指令是一组经常使用的指令，它包括移位指令（含算术移位指令，逻辑移位指令），循环移位指令（含带进位循环移位指令），其作用就是将目的操作数的所有位按操作符规定的方式移动指定的位数。

ARM 中的移位指令如下：ASR, LSR, LSL, ROR和 RRX，直接写入目标寄存器中。
<!-- more -->
`S`是一个可选的后缀，如果指定了 S 则根据操作会更新条件标志位。会被更新的标志位如下：

N：为 1 表示运算的结果为负数，为 0 表示运算的结果为正数。

Z：为 1 表示运算的结果为零，为 0 表示运算的结果不为零。

C：表示被移出的最后一位。

## ASR

叫做算术右移指令（Arithmetic shift right），从最左边开始依次往右移动，左端用第 31 位值填充，右端移动之后超出寄存器范围的位放入状态寄存器的 Carry Flag 位中。如下图所示：

![image-20220120150422054](https://cdn.jsdelivr.net/gh/wangjunstf/pics//images/image-20220120150422054.png)

## LSR

叫做逻辑右移指令（Logical shift right），从最左边开始依次往右移动，左端用 0 填充，右端移动之后超出寄存器范围的位放入状态寄存器的 Carry Flag 位中。如下图所示：

![image-20220120151139635](https://cdn.jsdelivr.net/gh/wangjunstf/pics//images/image-20220120151139635.png)

常用于除于 2 的 n 次幂，例如：

```
MOV R0,R1,LSR #4                ; R0 = R1/(2**4)        
```

## LSL 

叫做逻辑左移指令（Logical shift left），从右边开始依次向左移动，右边用 0 填充，左边超出寄存器范围的位放入状态寄存器的 Carry Flag 位中。如下图所示：

![image-20220120152429454](https://cdn.jsdelivr.net/gh/wangjunstf/pics//images/image-20220120152429454.png)

常用于乘于 2 的 n 次幂，例如：

```
MOV R0,R1,LSL #4               ; R0 = R1*(2**4)
```



## ROR

叫做循环右移指令（Rotate right），从左往右依次循环移动，左端用右端移出的位来填充，移入最左边的位同时也进入状态寄存器的 Carry Flag 位中。如下图所示：

![image-20220120153234827](https://cdn.jsdelivr.net/gh/wangjunstf/pics//images/image-20220120153234827.png)

## RRX 

叫做扩展的循环右移指令（Rotate right with extend），从左往右循环右移，最右边的位进入状态寄存器的 Carry Flag 位中，Carry Flag 位进入最左边的位中。如下图所示：

![image-20220120155205572](https://cdn.jsdelivr.net/gh/wangjunstf/pics//images/image-20220120155205572.png)

## 参考

https://developer.arm.com/documentation/dui0489/h/arm-and-thumb-instructions/shift-operations
