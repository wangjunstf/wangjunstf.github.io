---
title: Common Lisp函数
date: 2021-04-26 23:37:31
updated:
categories: [Lisp]
tags: [Lisp,函数]
---
# 定义新函数

新函数一般用defun宏来定义。其基本结构为：
(defun name
  "一些说明文字"
  body-form*)

任何符号都可以作为函数名，但通常函数名仅包含字典字符和连字符，但在特定的命名约定里，其它字符也允许使用。例如+函数可以将一个或多个数字相加。
<!-- more -->

```
(defun hello-world()
  "打印Hello,world"
  (format t "Hello,world"))
```

hello-world是函数名

* （）：形参列表，本函数无需传入实参
* "打印Hello,world" ：注释字符串，可以通过documentation函数来获取
  例如 ` (documentation 'hello-world 'function)`
* (format t "Hello,world") ：函数体，作用是向标准输出打印”Hello,world”

 body-form*可包含多个括号括起来的表达式，最后一个表达式的值作为函数的值返回。

# 函数形参列表

形参列表的基本用户为声明一些变量来接收传递给函数的实参，普通的形参列表，必须为每一个必要形参提供一个实参，不然会报错。

接下来讨论可选形参，剩余形参，关键字形参。

# 可选形参

当有时提供的实参数小于形参数时，未赋值的形参用默认值填充，默认值可以指定。在必要形参后放置符号&optional, 后接可选形参的名字。
例如：
`(defun foo (a b &optional c d) (list a b c d)`
可以这样调用：
(foo 1 2)  -> (1 2 NIL)
(foo 1 2 3) -> (1 2 3 NIL)
(foo 1 2 3 4) -> (1 2 3 4)

有时想为可选的形参名提供默人值，可以这样写：
`(defun foo (a b &optional (c 10)) (list a b c)`
(foo 1 2)  -> (1 2 10)
(foo 1 2 3) -> (1 2 3)

还可以基于其它形参来计算默认值
`(defun foo (a &optional (b a)) (list a b c)`
(foo 12)  -> (12 12)
(foo 12 6)  -> (12 6)

# 剩余形参

有时需要根据需要传递多个实参给形参，可能1个，2个或多个。
在必要形参，可选形参后放置&rest，那么可将满足了必要形参和可选形参之后的其余所有实参就会被被收集到一个列表里成为&rest形参的值。

`(defun + (&rest numbers) ...)`

# 关键字形参

假如调用者只想为四个参数中的一个提供值，或者更进一步，不同的调用者有可能将分别选择使用其中一个参数。
在任何必要的可选形参和剩余形参后放置符号&key以及任意数量的关键字形参标识符
`(defun foo (&key a b c) (list a b c) )`

(foo)     ->    (NIL,NIL,NIL)
(foo :a 1)   ->  (1,NIL,NIL)
(foo :b 1)   -> (NIL,1,NIL)
(foo :c 1)   -> (NIL,NIL,1)
(foo a: 1 :b 2 c: 3)    -> (1 2 3)
(foo a: 1 :c 3 b: 2)    -> (1 2 3)

# 混合不同的形参类型

各种形参的声明顺序必须是：必要形参，可选形参，然后剩余形参，最后关键字形参。
一般情况是把必要形参和另外一种类型的形参组合使用。
或者是：组合&optional形参和&rest形参。

如果一个函数要同时使用&optional形参和&key形参，应该把所有形参都是用&key形参，这样更加灵活，并且总可以在不破坏该函数的已有调用的情况下添加新的关键字形参。

一般而言，使用关键字形参会使代码相对易于维护和扩展。

# 函数返回值

默认情况下函数最后一个表达式的值作为整个函数的返回值。

也可以使用return-from语句来在任何位置返回。

例如：
该函数用来发现一个数对，其中每个数都小于10，并且其乘机大于函数的参数n。

```
(defun foo (n)
  (dotimes (i 10)
    (dotimes (j 10)
      (when (> (* i j) n)
	      (return-from foo (list i j))))))
```


# 作为数据的函数——高阶函数

函数也可以作为一种数据，保存在变量里，还可以传递给其它函数。
用defun定义一个函数时，实际上做了两件事：创建一个新的函数对象以及赋予其一个名字，也可以用lambda表达式来创建一个没有名字的函数。

一个函数对象的实际表示，无论是有名字还是匿名的，都只是一些二进制数据——以原生编译的Lisp形式存在，可能大部分是由机器码构成。

只需要知道如何保持它们，和需要时如何调用它们。

特殊操作符function可以用来获取一个函数对象
num++为函数名，用来将函数参数递增1

```
CL-USER> (defun num++(n) (+ n 1))
NUM++
CL-USER> (num++ 12)
13
CL-USER> (num++ 14)
15
CL-USER> (function num++)
#<FUNCTION NUM++>     ;NUM++为函数对象
```

 (function num++) 类似于 \#‘num++

当得到了函数对象，就可以调用它

可以通过funcall函数和apply函数来通过函数对象调用函数。
两者的区别：

* funcall用于在编写代码时确切知道传递给函数多少个实参
* (apply 'num++ ‘(1))  类似于 (funcall \#’num++ 1)

# 匿名函数

使用lambda创建匿名函数的形式如下：
`(lambda (parameters) body)`

例如:
`(funcall #'(lambda (x y) (+ x y)) 2 3)  ;返回5