---
title: LR语法分析
date: 2021-05-15 16:48:32
updated:
categories: [编译原理]
tags: [编译原理,LR分析]
---
## LR分析

### 分析过程

* **三元式表示**
  三元式子：（状态栈，符号栈，输入符号串）
<!-- more -->
* 初始时，将状态s<sub>0</sub>和#进分析栈，三元式为：

  (S<sub>0</sub>，#，a<sub>1</sub>a<sub>2</sub>...a<sub>n</sub>#)

* 任意时刻三元式：

  (S<sub>0</sub>S<sub>1</sub>...==S<sub>m</sub>==，#X<sub>1</sub>X<sub>2</sub>...X<sub>n</sub>，==a<sub>1</sub>==a<sub>2</sub>...a<sub>n</sub>#)

  ==分析器的下一步动作是由栈顶状态S<sub>m</sub>和输入符号串a<sub>i</sub>唯一确定==

  通过查询action表可以确定下一个状态，之后会讨论怎么构建action表。



### 下一步动作有四种情况

**移进**：将当前符号移进符号栈，相应的状态移进状态栈

**规约**：满足产生式时，右端长度为r，则两个栈顶的r个元素同时出栈，将产生式左端符号进入符号栈，**根据此时状态栈栈顶符号和符号栈栈顶符号确定下一步状态**

**接受**：分析成功

**出错**：报告出错信息



### LR文法

从给定的文法构造一个识别该文法前缀的确定有限状态自动机(DFA)，根据该DFA构造分析表。



### 活前缀

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8A%E5%8D%8811.26.36.png" alt="截屏2021-04-20 上午11.26.36" style="zoom: 33%;" />



从左到右为最左推导，其逆过程为最右规约。

从右到左为最右推导（==规范推导==），其逆过程为最左规约，也叫(==规范规约==)



<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8A%E5%8D%8811.36.22.png" alt="截屏2021-04-20 上午11.36.22" style="zoom: 33%;" />

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8A%E5%8D%8811.17.41.png" alt="截屏2021-04-20 上午11.17.41" style="zoom: 33%;" />



**活前缀的作用：**

只要输入串已扫描部分保持可规约成一个活前缀，那就意味着所扫描过的部分没有错误。



## LR(0)项目及其项目集规范族

### 定义

对于文法G，分别为其产生式右部的每个字符的左右两边添加特殊符号"."，就构成文法的一个LR(0)项目



例如，现有一个文法G：

S $\rightarrow$ aAcBe

其LR(0)项目共有6个，分别为：

S $\rightarrow$ .aAcBe

S $\rightarrow$ a.AcBe

S $\rightarrow$ aA.cBe

S $\rightarrow$ aAc.Be

S $\rightarrow$ aAcB.e

S $\rightarrow$ aAcBe.



### 项目分类

根据"."所在的位置，可将项目分为：

1. "."之后为终结符，称为**移进项目**。

2. "."之后为非终结符，称为**待约项目**

   意思是"."之后的非终结符，将用于输入符号的规约

3. "."之后没有符号，称为**规约项目**

4. 对于拓广文法S' $\rightarrow$ S，若此时"."位于S之后，则代表分析结束。该项目称为**接受项目**



### 构造识别活前缀的NFA和DFA

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.31.23.png" alt="截屏2021-04-20 下午4.31.23" style="zoom: 33%;" />

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.33.05.png" alt="截屏2021-04-20 下午4.33.05" style="zoom: 33%;" />

状态1碰到ε可以跳转到状态3和状态11，因为其后为非终结符，可以直接跳到该终结符所对应的状态



<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.38.42.png" alt="截屏2021-04-20 下午4.38.42" style="zoom:33%;" />



## LR(0)文法

若一个文法G的项目集规范族中的所有项目都是相容的，则称文法G为LR(0)文法

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.41.32.png" alt="截屏2021-04-20 下午4.41.32" style="zoom:33%;" />



<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.42.23.png" alt="截屏2021-04-20 下午4.42.23" style="zoom:25%;" />

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.41.32.png" alt="截屏2021-04-20 下午4.41.32" style="zoom: 25%;" />

### LR(0)文法示例

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.42.23.png" alt="截屏2021-04-20 下午4.42.23" style="zoom:25%;" />



1）

G[S]拓广为：<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.47.35.png" alt="截屏2021-04-20 下午4.47.35" style="zoom:33%;" />

画出识别活前缀的DFA

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.48.18.png" alt="截屏2021-04-20 下午4.48.18" style="zoom: 25%;" />

2）

构造LR(0)分析表

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.49.17.png" alt="截屏2021-04-20 下午4.49.17" style="zoom: 33%;" />



3）用LR分析法对对abbce进行分析

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%884.50.37.png" alt="截屏2021-04-20 下午4.50.37" style="zoom:33%;" />

