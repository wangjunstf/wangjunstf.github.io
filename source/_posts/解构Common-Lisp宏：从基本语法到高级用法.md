---
title: 解构Common Lisp宏：从基本语法到高级用法
date: 2025-01-16 18:33:41
updated:
categories: Lisp
tags: [Common Lisp,宏,WHEN,COND,DO,DOLIST,DOTIMES]
---
其余自Lisp的许多编程思想，从条件表达式到垃圾收集，都已经被吸取进其他语言，但Lisp的宏系统却始终使它保持了在语言风格上的独特性。Lisp的宏和大多数其他语言中的也叫宏的东西是完全不一样的，要完全认识Lisp中的宏系统，就需要重新看待它。
<!-- more -->
一种常见的观点认为，语言的定义可能包含一个使用“核心”语言实现的标准功能库——如果某些功能没有被实现在标准库中，那么他们可能被程序员使用标准库提供的功能实现了。C的标准库差不多可以完全用可移植的C来实现，类似的，Java的标准Java开发包(JDK)中提供的不断改进的类和接口也是用"纯"Java编写的。

使用核心加上标准库的方式来定义语言的优势在于易于理解和实现，但真正的好处是很容易对其进行扩展，如果C语言中不包括做某件事的函数，那么就可以写出这个函数。类似的，类似Java这样的语言通过定义新的类就可以扩展该语言。

**尽管Common Lisp支持所有这些扩展语言的方法，但宏还提供了另一个表达方法。** 每个宏都有自己的语法，它们能够决定那些被传递的S-表达式如何转换成Lisp形式。核心语言有了宏就可以构造出新的语法，例如WHEN, DOLIST和LOOP这样的控制构造以及DEFUN和DEFPARAMETER这样的定义形式，从而使这些新语法可以作为“标准库”的一部分而不是将其硬编码到语言核心。这已经涉及到语言本身是如何实现的，但作为一个Lisp程序员，我们更关心的是它所提供的语言扩展方式，而这将使Common Lisp成为更好的用于表达特定编程问题解决方案的语言。

**S-表达式**
S-表达式的基本元素是列表(lisp)和原子(atom)，列表由括号所包围，并可包含任何数量的由空格所分割的元素。原子是除了列表之外的所有元素，包括符号、数字、字符串等。列表元素本身也可以是S-表达式(原子或嵌套的列表)

**作为Lisp形式的S-表达式**
在读取器把大量文本转化为S-表达式后，这些S-表达式随后可以作为Lisp形式被求值，并不是每个读取器可读的S-表达式都有必要作为Lisp形式来求值，Common Lisp的求值规则定义了第二层的语法来检测哪种S-表达式可以看作Lisp形式。例如，从读取 (foo 1 2) 得到的S-表达式在句法上是良好定义的，但是只有当foo是一个函数或宏的名字时，它才可以被求值。

现在以Lisp中常见的几个宏开始讲解，它们都是如果Lisp没有宏，就必须构造在语言核心里的东西。

## 一 WHEN 和 UNLESS
最基本的条件执行形式是由IF特殊操作符提供的，其基本形式是：如果condition成立，那么执行then-form，否则执行else-form。
```
(if condition then-form [else-form])
```
其中then-form或else-form必须是单一的Lisp形式，如果要在每个子句中执行一系列的操作，则必须将其用其他一些语法形式进行封装，例如：
```
(if (condition)
	(progn 
		(then-form1)
		(then-form2)))
```
类似上述这样的代码可以实现，当condition为真时，做这个，那个以及一些事情。可否有一种方式能将这种IF加上PROGN所组成的模式抽象出来，宏就能实现。Common Lisp提供了一个标准宏WHEN，可以这么写：
```
(when  (condition)
	(then-form1)
	(then-form2))
```
如果WHEN没有被内置到标准库中，我们可以使用下面这个宏来定义自己的WHEN：
```
(defmacro when (condition &rest body)
	`(if ,condition (progn ,@body)))
```
与WHEN宏同系列的另一个宏是UNLESS，它取相反的条件，只有当条件为假时才求值其形式体：
```
(defmacro unless (condition &rest body)
	`(if (not ,condition) (progn ,@body)))
```
必须承认，这些都是相当简单的宏，它们只是抽象掉了一些语言层面约定俗成的细节，其实宏可以被更大规模地用于创建完整的特定领域的嵌入式语言。

反引号（\`）用于 宏展开（macro expansion）时，它的作用是对表达式进行部分求值，以便在表达式中直接引用变量或常量，同时保持部分结构不变。
逗号 `,`表示对表达式进行求值
`,`（unquote）会将一个表达式求值并插入到宏扩展的代码中。

```
(defmacro example1 (x)
  `(list 1 2 ,x 4))  ; 使用 , 来插入 x 的值

(example1 3)  ; 执行宏，传入 3
```
展开后，`example1` 会将 `x` 的值（即 3）插入到 `list` 中，结果如下：
```
; 展开后的代码 
(list 1 2 3 4)
```

` ,@`是 解包操作符（unquote-splicing），它用于在列表中“展开”或“解包”一个列表，使其中的元素被直接插入到外部列表中。
```
(defmacro example2 (x)
  `(list 1 2 ,@x 4))  ; 使用 ,@ 来展开 x

(example2 '(3 4 5))  ; 执行宏，传入 (3 4 5)

```
在这个例子中，`x` 是一个列表 `(3 4 5)`，使用 `,@` 会将其元素 `3`、`4` 和 `5` 解包并插入到 `list` 中，结果如下：
```
; 展开后的代码
(list 1 2 3 4 5 4)
```
## 二 COND
当遇到多重分支的条件语句时，原始的IF表达式再次变得丑陋不堪：如果a成立那么执行x，否则如果b成立那么执行y，否则执行z：
```
(if a
	(do-x)
	(if b
		(do-y)
		(do-z)))
```
如果需要在then子句中包括多个形式，就需要用到PROGN，代码结构会变得更加复杂。因此 Common Lisp提供了用于表达多重分支条件的宏 COND。
```
(cond
  (test1 expr1)
  (test2 expr2)
  ...
  (t exprN))  ; 't' 是默认情况，类似于 else
```
在 `cond` 语句中，Lisp 会逐个评估每个条件（`test1`, `test2` 等），直到找到一个结果为真（非 `nil`）的条件。如果找到了真条件，Lisp 会执行对应的表达式（`expr1`, `expr2` 等），并返回其结果。`t` 表示“真”，如果没有条件匹配，`t` 分支会执行。

## 三 AND，OR 和 NOT
NOT是函数，它接受单一参数并对其值取反，当参数为NIL时返回T，否则返回NIL。
AND和OR则是宏，它们实现了对任意数量子表达式的逻辑合取和析取操作，并被定义成宏以支持“短路”特性。

只要AND的一个子表达式求值为NIL，它就立即停止并返回NIL，如果将所有子表达式都求值为非NIL，那么它将返回最后一个子表达式的值。

对于OR来说，只要一个子表达式求值得到非NIL，它就立即停止并返回当前子表达式的值，如果没有子表达式求值为真，OR返回NIL。
```
(not nil)             ; --> T
(not (= 1 1))         ; --> NIL
(and (= 1 2) (= 3 3)) ; --> NIL
(or (= 1 2) (= 3 3))  ; --> T
```
## 四 循环
循环构造是另外一类主要的控制构造。Common Lisp的循环机制，除了更加强大和灵活以外，还是一门关于宏所提供的“鱼和熊掌兼得”的编程风格的有趣课程。
DO提供了一种基本的结构化循环构造，而DOLIST和DOTIMES则提供了两种易用却不那么通用的构造。最后，LOOP宏提供了一种成熟的微型语言，它是一种独特的表达循环的方式。不论你喜欢或不喜欢这种循环的构造方式，LOOP本身都是为语言增加新构造的宏展示其强大威力的突出示例。

## 五 DOLIST 和 DOTIMES
先从易于使用的DOLIST和DOTIMES宏开始。
DOLIST在一个列表的元素上循环操作，使用一个依次持有列表中所有后继元素的变量来执行循环体。下面是其基本形式（去掉了一些难懂的选项）
```
(dolist (var list-form)
	body-form*)
```
例如：
```
CL-USER> (dolist (x '(1 2 3)) (print x))
1 
2 
3 
NIL
```
在这种方式下，DOLIST这种形式本身求值为NIL。
如果想在列表结束之前中断一个DOLIST循环，则可以使用RETURN。
```
CL-USER> (dolist (x '(1 2 3)) (print x) (if (evenp x) (return)))
1 
2 
NIL
```
evenp 判断一个数是否为偶数。

DOTIMES是用于循环计数的高级循环构造，其基本模板和DOLIST非常相似。
```
(dotimes (var count-form)
	body-form*)
```
其中count-form必须能求值为一个整数，通过每次循环，var所持有的整数依次从0到比那个数小1的每一个后继整数。
```
CL-USER> (dotimes (i 4) (print i))
0 
1 
2 
3 
NIL
```

## 六 DO
尽管DOLIST和DOTIMES方便易用，但也无法应用于所有循环，使用DO可以解决所有的循环问题。与DOLIST和DOTIMES只提供一个循环变量有所不同的是，DO允许绑定任意数量的变量，并且变量值在每次循环中的改变也是可自定义的，也可以自定义测试条件来决定何时终止循环，并可以提供一个形式，在循环结束时进行求值来为DO表达式整体生成一个返回值。
```
(do (variable-definition*)
	(end-test-form result-form*)
	statement*)
```
每一个variable-definition引入一个将存在于循环体作用域之内的变量，单一变量定义的完整形式是含有三个元素的列表。
```
(var init-form step-form)
```
上述init-form在循环开始时被求值并将结果值绑定到变量var上，在循环的每一个后续迭代开始之前，step-form将被求值并把新值赋值给var，step-form是可选的。

在每次迭代开始时以及所有循环变量都被指定新值后，end-test-form会被求值，只要其值为NIL，迭代过程就会继续。

当end-test-form求值为真，result-form将被求值，且最后一个结果形式的值将作为DO表达式的值返回。
示例：计算斐波那契数列的第 n 项
```
(do ((n 0 (1+ n))
     (cur 0 next)
     (next 1 (+ cur next)))
    ((= 4 n) cur))
```
示例：打印前N个自然数的值：
```
(do ((i 0 (1+ i)))
    ((>= i 4))
  (print i))
```
当然打印前N个自然数的值使用DOTIMES会更加方便：
```
(dotimes (i 4) (print i))
```

下面的例子演示一个不绑定循环变量的DO循环，在当前时间小于一个全局变量值的时候，它保持循环，每分钟打印一个"Waiting"：
```
(do ()
	((> (get-universal-time) *some-future-data*))
	(format t "Waiting~%")
	(sleep 60))
```

## 七 强大的LOOP
简单的情形可以使用DOLIST和DOTIMES，如果它们不满足要求可以使用通用的DO。但是有一种循环需求在实际开发中很常见，例如在多种数据结构上的循环：列表，向量，哈希表和包，或者在循环时以多种方式来聚集值：收集，计数，求和，最小化，最大化等这一系列问题都可以用LOOP宏轻松实现。
LOOP宏有两大版本，简化的扩展的。简化版是没有变量绑定的无限循环，而扩展版则允许复杂的变量绑定和条件判断。
```
(loop
	body-form*)
```
主体形式在每次循环时都被求值，整个循环将不停的迭代，直到使用RETURN来进行终止。例如可以使用一个简化的LOOP来写出前面的DO循环。
```
(loop
	(when (> (get-universal-time) *some-future-data*) 
		(return))
	(format t "Waiting ~%")
	(sleep 60))
```
问题，将1到10的数字收集到一个列表中：
使用DO循环：
```
(do ((nums nil) (i 1 (1+ i)))
    ((> i 10) (nreverse nums))
  (push i nums))
```
使用LOOP：
```
(loop for i from 1 to 10 collecting i)
```
关于LOOP的这种语法，有些人喜欢，有些人则讨厌。

下面是关于LOOP的其他一些用法：
1. 统计一个字符串中元音字母的个数：
```
(loop for  x across "This is my blog" counting (find x "aeiou"))
```
2. 计算第10个斐波那契数，它类似于前面的DO循环版本：
```
(loop for i below 10
      and a = 0 then b
      and b = 1 then (+ b a)
      finally (return a))
```
符号across, and, below, collecting, counting, finally, for, from, summing, then 和to都是一些循环关键字。值得注意的是，我们通过LOOP可以看到，宏是如何被用于扩展基本语言的。尽管LOOP提供了它自己的语法用来表达循环构造，但它并没有抹杀Lisp的其他优势，如果它没有被包括在标准库中，你也可以自己实现或借助一个第三方库来实。现其余自Lisp的许多编程思想，从条件表达式到垃圾收集，都已经被吸取进其他语言，但Lisp的宏系统却始终使它保持了在语言风格上的独特性。Lisp的宏和大多数其他语言中的也叫宏的东西是完全不一样的，要完全认识Lisp中的宏系统，就需要重新看待它。

一种常见的观点认为，语言的定义可能包含一个使用“核心”语言实现的标准功能库——如果某些功能没有被实现在标准库中，那么他们可能被程序员使用标准库提供的功能实现了。C的标准库差不多可以完全用可移植的C来实现，类似的，Java的标准Java开发包(JDK)中提供的不断改进的类和接口也是用"纯"Java编写的。

使用核心加上标准库的方式来定义语言的优势在于易于理解和实现，但真正的好处是很容易对其进行扩展，如果C语言中不包括做某件事的函数，那么就可以写出这个函数。类似的，类似Java这样的语言通过定义新的类就可以扩展该语言。

**尽管Common Lisp支持所有这些扩展语言的方法，但宏还提供了另一个表达方法。** 每个宏都有自己的语法，它们能够决定那些被传递的S-表达式如何转换成Lisp形式。核心语言有了宏就可以构造出新的语法，例如WHEN, DOLIST和LOOP这样的控制构造以及DEFUN和DEFPARAMETER这样的定义形式，从而使这些新语法可以作为“标准库”的一部分而不是将其硬编码到语言核心。这已经涉及到语言本身是如何实现的，但作为一个Lisp程序员，我们更关心的是它所提供的语言扩展方式，而这将使Common Lisp成为更好的用于表达特定编程问题解决方案的语言。

**S-表达式**
S-表达式的基本元素是列表(lisp)和原子(atom)，列表由括号所包围，并可包含任何数量的由空格所分割的元素。原子是除了列表之外的所有元素，包括符号、数字、字符串等。列表元素本身也可以是S-表达式(原子或嵌套的列表)

**作为Lisp形式的S-表达式**
在读取器把大量文本转化为S-表达式后，这些S-表达式随后可以作为Lisp形式被求值，并不是每个读取器可读的S-表达式都有必要作为Lisp形式来求值，Common Lisp的求值规则定义了第二层的语法来检测哪种S-表达式可以看作Lisp形式。例如，从读取 (foo 1 2) 得到的S-表达式在句法上是良好定义的，但是只有当foo是一个函数或宏的名字时，它才可以被求值。

现在以Lisp中常见的几个宏开始讲解，它们都是如果Lisp没有宏，就必须构造在语言核心里的东西。

## 一 WHEN 和 UNLESS
最基本的条件执行形式是由IF特殊操作符提供的，其基本形式是：如果condition成立，那么执行then-form，否则执行else-form。
```
(if condition then-form [else-form])
```
其中then-form或else-form必须是单一的Lisp形式，如果要在每个子句中执行一系列的操作，则必须将其用其他一些语法形式进行封装，例如：
```
(if (condition)
	(progn 
		(then-form1)
		(then-form2)))
```
类似上述这样的代码可以实现，当condition为真时，做这个，那个以及一些事情。可否有一种方式能将这种IF加上PROGN所组成的模式抽象出来，宏就能实现。Common Lisp提供了一个标准宏WHEN，可以这么写：
```
(when  (condition)
	(then-form1)
	(then-form2))
```
如果WHEN没有被内置到标准库中，我们可以使用下面这个宏来定义自己的WHEN：
```
(defmacro when (condition &rest body)
	`(if ,condition (progn ,@body)))
```
与WHEN宏同系列的另一个宏是UNLESS，它取相反的条件，只有当条件为假时才求值其形式体：
```
(defmacro unless (condition &rest body)
	`(if (not ,condition) (progn ,@body)))
```
必须承认，这些都是相当简单的宏，它们只是抽象掉了一些语言层面约定俗成的细节，其实宏可以被更大规模地用于创建完整的特定领域的嵌入式语言。

反引号（\`）用于 宏展开（macro expansion）时，它的作用是对表达式进行部分求值，以便在表达式中直接引用变量或常量，同时保持部分结构不变。
逗号 `,`表示对表达式进行求值
`,`（unquote）会将一个表达式求值并插入到宏扩展的代码中。

```
(defmacro example1 (x)
  `(list 1 2 ,x 4))  ; 使用 , 来插入 x 的值

(example1 3)  ; 执行宏，传入 3
```
展开后，`example1` 会将 `x` 的值（即 3）插入到 `list` 中，结果如下：
```
; 展开后的代码 
(list 1 2 3 4)
```

` ,@`是 解包操作符（unquote-splicing），它用于在列表中“展开”或“解包”一个列表，使其中的元素被直接插入到外部列表中。
```
(defmacro example2 (x)
  `(list 1 2 ,@x 4))  ; 使用 ,@ 来展开 x

(example2 '(3 4 5))  ; 执行宏，传入 (3 4 5)

```
在这个例子中，`x` 是一个列表 `(3 4 5)`，使用 `,@` 会将其元素 `3`、`4` 和 `5` 解包并插入到 `list` 中，结果如下：
```
; 展开后的代码
(list 1 2 3 4 5 4)
```
## 二 COND
当遇到多重分支的条件语句时，原始的IF表达式再次变得丑陋不堪：如果a成立那么执行x，否则如果b成立那么执行y，否则执行z：
```
(if a
	(do-x)
	(if b
		(do-y)
		(do-z)))
```
如果需要在then子句中包括多个形式，就需要用到PROGN，代码结构会变得更加复杂。因此 Common Lisp提供了用于表达多重分支条件的宏 COND。
```
(cond
  (test1 expr1)
  (test2 expr2)
  ...
  (t exprN))  ; 't' 是默认情况，类似于 else
```
在 `cond` 语句中，Lisp 会逐个评估每个条件（`test1`, `test2` 等），直到找到一个结果为真（非 `nil`）的条件。如果找到了真条件，Lisp 会执行对应的表达式（`expr1`, `expr2` 等），并返回其结果。`t` 表示“真”，如果没有条件匹配，`t` 分支会执行。

## 三 AND，OR 和 NOT
NOT是函数，它接受单一参数并对其值取反，当参数为NIL时返回T，否则返回NIL。
AND和OR则是宏，它们实现了对任意数量子表达式的逻辑合取和析取操作，并被定义成宏以支持“短路”特性。

只要AND的一个子表达式求值为NIL，它就立即停止并返回NIL，如果将所有子表达式都求值为非NIL，那么它将返回最后一个子表达式的值。

对于OR来说，只要一个子表达式求值得到非NIL，它就立即停止并返回当前子表达式的值，如果没有子表达式求值为真，OR返回NIL。
```
(not nil)             ; --> T
(not (= 1 1))         ; --> NIL
(and (= 1 2) (= 3 3)) ; --> NIL
(or (= 1 2) (= 3 3))  ; --> T
```
## 四 循环
循环构造是另外一类主要的控制构造。Common Lisp的循环机制，除了更加强大和灵活以外，还是一门关于宏所提供的“鱼和熊掌兼得”的编程风格的有趣课程。
DO提供了一种基本的结构化循环构造，而DOLIST和DOTIMES则提供了两种易用却不那么通用的构造。最后，LOOP宏提供了一种成熟的微型语言，它是一种独特的表达循环的方式。不论你喜欢或不喜欢这种循环的构造方式，LOOP本身都是为语言增加新构造的宏展示其强大威力的突出示例。

## 五 DOLIST 和 DOTIMES
先从易于使用的DOLIST和DOTIMES宏开始。
DOLIST在一个列表的元素上循环操作，使用一个依次持有列表中所有后继元素的变量来执行循环体。下面是其基本形式（去掉了一些难懂的选项）
```
(dolist (var list-form)
	body-form*)
```
例如：
```
CL-USER> (dolist (x '(1 2 3)) (print x))
1 
2 
3 
NIL
```
在这种方式下，DOLIST这种形式本身求值为NIL。
如果想在列表结束之前中断一个DOLIST循环，则可以使用RETURN。
```
CL-USER> (dolist (x '(1 2 3)) (print x) (if (evenp x) (return)))
1 
2 
NIL
```
evenp 判断一个数是否为偶数。

DOTIMES是用于循环计数的高级循环构造，其基本模板和DOLIST非常相似。
```
(dotimes (var count-form)
	body-form*)
```
其中count-form必须能求值为一个整数，通过每次循环，var所持有的整数依次从0到比那个数小1的每一个后继整数。
```
CL-USER> (dotimes (i 4) (print i))
0 
1 
2 
3 
NIL
```

## 六 DO
尽管DOLIST和DOTIMES方便易用，但也无法应用于所有循环，使用DO可以解决所有的循环问题。与DOLIST和DOTIMES只提供一个循环变量有所不同的是，DO允许绑定任意数量的变量，并且变量值在每次循环中的改变也是可自定义的，也可以自定义测试条件来决定何时终止循环，并可以提供一个形式，在循环结束时进行求值来为DO表达式整体生成一个返回值。
```
(do (variable-definition*)
	(end-test-form result-form*)
	statement*)
```
每一个variable-definition引入一个将存在于循环体作用域之内的变量，单一变量定义的完整形式是含有三个元素的列表。
```
(var init-form step-form)
```
上述init-form在循环开始时被求值并将结果值绑定到变量var上，在循环的每一个后续迭代开始之前，step-form将被求值并把新值赋值给var，step-form是可选的。

在每次迭代开始时以及所有循环变量都被指定新值后，end-test-form会被求值，只要其值为NIL，迭代过程就会继续。

当end-test-form求值为真，result-form将被求值，且最后一个结果形式的值将作为DO表达式的值返回。
示例：计算斐波那契数列的第 n 项
```
(do ((n 0 (1+ n))
     (cur 0 next)
     (next 1 (+ cur next)))
    ((= 4 n) cur))
```
示例：打印前N个自然数的值：
```
(do ((i 0 (1+ i)))
    ((>= i 4))
  (print i))
```
当然打印前N个自然数的值使用DOTIMES会更加方便：
```
(dotimes (i 4) (print i))
```

下面的例子演示一个不绑定循环变量的DO循环，在当前时间小于一个全局变量值的时候，它保持循环，每分钟打印一个"Waiting"：
```
(do ()
	((> (get-universal-time) *some-future-data*))
	(format t "Waiting~%")
	(sleep 60))
```

## 七 强大的LOOP
简单的情形可以使用DOLIST和DOTIMES，如果它们不满足要求可以使用通用的DO。但是有一种循环需求在实际开发中很常见，例如在多种数据结构上的循环：列表，向量，哈希表和包，或者在循环时以多种方式来聚集值：收集，计数，求和，最小化，最大化等这一系列问题都可以用LOOP宏轻松实现。
LOOP宏有两大版本，简化的扩展的。简化版是没有变量绑定的无限循环，而扩展版则允许复杂的变量绑定和条件判断。
```
(loop
	body-form*)
```
主体形式在每次循环时都被求值，整个循环将不停的迭代，直到使用RETURN来进行终止。例如可以使用一个简化的LOOP来写出前面的DO循环。
```
(loop
	(when (> (get-universal-time) *some-future-data*) 
		(return))
	(format t "Waiting ~%")
	(sleep 60))
```
问题，将1到10的数字收集到一个列表中：
使用DO循环：
```
(do ((nums nil) (i 1 (1+ i)))
    ((> i 10) (nreverse nums))
  (push i nums))
```
使用LOOP：
```
(loop for i from 1 to 10 collecting i)
```
关于LOOP的这种语法，有些人喜欢，有些人则讨厌。

下面是关于LOOP的其他一些用法：
1. 统计一个字符串中元音字母的个数：
```
(loop for  x across "This is my blog" counting (find x "aeiou"))
```
2. 计算第10个斐波那契数，它类似于前面的DO循环版本：
```
(loop for i below 10
      and a = 0 then b
      and b = 1 then (+ b a)
      finally (return a))
```
符号across, and, below, collecting, counting, finally, for, from, summing, then 和to都是一些循环关键字。值得注意的是，我们通过LOOP可以看到，宏是如何被用于扩展基本语言的。尽管LOOP提供了它自己的语法用来表达循环构造，但它并没有抹杀Lisp的其他优势，如果它没有被包括在标准库中，你也可以自己实现或借助一个第三方库来实现。