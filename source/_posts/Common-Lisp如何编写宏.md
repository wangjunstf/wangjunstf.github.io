---
title: Common Lisp如何编写宏
date: 2025-01-16 18:41:56
updated:
categories: Lisp
tags: [Common Lisp, 宏]
---
## 一 宏展开期和运行期
编写宏就是在编写那些将被编译器用来生成代码并随后编译的程序，只有当所有宏都被完全展开并且产生的代码被编译后，程序才可以实际运行。宏运行的时期被称为宏展开期(macro expansion time)，这和运行期(runtime)是不同的，宏展开期无法访问运行期的数据。
<!-- more -->
## 二 DEFMACRO
宏使用DEFMACRO来定义，其基本框架为：
```
(defmacro name (parameter*)
	"Optional documentation string."
	body-form*)
```
和函数一样，宏由名字，形参列表，可选的文档字符串以及Lisp表达式体所构成。宏的工作是将宏形式转化成做特定事情的代码。

对于简单的宏，编写一个反引用模板并将宏参数差人到正确的位置。复杂的宏则是一个庞大的独立程序，它将带有配套的助手函数和数据结构。编写完宏以后需要确保它所提供的抽象没有“泄露”其实现细节。

总结起来，编写宏的步骤如下：
（1）辨析示例的宏调用以及它应当展开生成的代码，反之亦然。
（2）编写从示例调用的参数中生成手写展开式的代码。
（3）确保宏抽象不产生“泄露”。

## 三 示例宏：do-primes
do-primes 宏类似于DOTIMES和DOLIST的循环构造，只是它并非迭代在整数或者一个列表的元素上，而是迭代在相继的素数上。
需要两个工具函数：一个用来测试给定的数是否为素数，另一个用来返回大于或等于其实参的下一个素数。
```
(defun primep (number)
  (when (> number 1)
    (loop for fac from 2 to (isqrt number) never (zerop (mod number fac)))))

(defun next-prime (number)
  (loop for n from number when (primep n) return n))
```
我们首先需要编写一个宏调用示例：
```
(do-primes (p 0 19)
	(format t "~d" p))
```
这个循环在每个大于等于0并小于等于19的素数上依次执行循环体。如果没有do-primes宏，我们可以使用DO来实现：
```
(do ((p (next-prime 0) (next-prime (1+ p))))
    ((> p 19))
  (format t "~d " p))
```
但上述代码不惧有通用性，使用宏do-primes，我们可以自定义循环体。

## 四 宏形参
任何宏的第一步工作都是提取出那些对象中用于计算展开式的部分，对于简单的宏，只需定义正确的形参来保存不同的实参就可以。
```
(do-primes (p 0 19)
	(format t "~d" p))
```
从上述宏调用可以看到，do-primes的第一个参数是一个列表，其含有循环变量的名字P以及下界0和上界19，第二个参数是format打印。
```
(defmacro do-primes ((var start end) &body body)
  `(do ((,var (next-prime ,start) (next-prime (1+ ,var))))
       ((> ,var ,end))
     ,@body))
```
上述`(var start end)`是所谓的解构式参数列表，“解构”涉及分拆一个结构体。
以上宏可以如下调用：其中p相当于是var, 0相当于是start, 19相当于是end。
```
CL-USER> (do-primes (p 0 19) (format t "~d " p))
2 3 5 7 11 13 17 19 
NIL
```

宏形参列表的另一个特性是可以使用&body作为&rest的同名词，它们在语义上是等价的，&body被用来保存一个构成该宏主体的形式的列表。
```
CL-USER> (macroexpand-1 '(do-primes (p 0 19) (format t "~d " p)))
(DO ((P (NEXT-PRIME 0) (NEXT-PRIME (1+ P)))) ((> P 19)) (FORMAT T "~d " P))
T
```
使用`macroexpand-1`可以查看宏调用产生的代码。也可以通过快捷键查看宏的展开式，光标移动到宏调用的行尾，输入 `C-c RET`

## 五 生成展开式
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
## 六 堵住漏洞
### 6.1 多重求值漏洞
假设我们没有使用19这样的字面数字，而是用(random 10)这样的表达式在end的位置上来调用do-primes：
```
(do-primes (p 0 (random 1000))
	(format t "~d " p))
```
它生成的代码如下：
```
(DO ((P (NEXT-PRIME 0) (NEXT-PRIME (1+ P))))
    ((> P (RANDOM 100)))
  (FORMAT T "~d " P))
```
它会在每次迭代时都执行` (RANDOM 100)`，显然是不对的。为了堵住上述漏洞，我们可以定义一个变量 ending-value 来保存`(RANDOM 100)`的值。
```
(defmacro do-primes ((var start end) &body body)
  `(do ((ending-value ,end)
	(,var (next-prime ,start) (next-prime (1+ ,var))))
       ((> ,var ending-value))
     ,@body))
```
### 6.2 数值依赖漏洞
上述的修改又引入了一个新漏洞，在以下调用中我们希望 end 参数是 start 的两倍。如果 end 的计算（即 (* 2 start)）在 var 初始化之前进行，那么它将能够正确地使用 start 的值。但如果 end 的计算在 var 初始化之后进行，而 var 的初始化又以某种方式依赖于 start 的值，这可能会导致问题。
```
(let ((start 10))
  (do-primes (p start (* 2 start))
    (format t "~d " p)))
```
正确的版本应该是将`ending-value`的定义放到后面：
```
(defmacro do-primes ((var start end) &body body)
  `(do ((,var (next-prime ,start) (next-prime (1+ ,var)))
	(ending-value ,end))
       ((> ,var ending-value))
     ,@body))
```
### 6.3 命名漏洞
最后一个需要堵上的漏洞是由于使用了变量名ending-value而产生的，假设宏调用如下：
```
(do-primes (ending-value 0 10)
	(print ending-value))
```
传递给do-primes的名字ending-value和宏体内的ending-value产生了干扰，为了解决这个问题，应该在宏体内使用一个罕见的名字，确保不会和调用者所传入的相同。

函数 GENSYM在其每次调用时返回唯一的符号，该符号在全局范围内唯一，避免用户代码中可能存在的变量名冲突，这样可以确保每次do-primes被展开时生成一个新符号以替代像ending-value这样的字面名称。
```
(defmacro do-primes ((var start end) &body body)
  (let ((ending-value-name (gensym)))
    `(do ((,var (next-prime ,start) (next-prime (1+ ,var)))
	(,ending-value-name ,end))
       ((> ,var ,ending-value-name))
       ,@body)))
```
当我们这样调用时：
```
(do-primes (p 0 (random 100))
  (format t "~d " p))
```
展开式为：
```
(DO ((P (NEXT-PRIME 0) (NEXT-PRIME (1+ P)))
     (#:G323 (RANDOM 100)))
    ((> P #:G323))
  (FORMAT T "~d " P))
```
#:G323 在每次宏展开时都不一样，这样能确保全局范围内都不会同名。

## 七 用于编写宏的宏
宏的作用是将常见的句法模式抽象掉，例如在最后版本的do-primes，它们都以一个LET形式开始，它引入了一些变量用来保存宏展开过程中用到的生成符号，我们可以用一个宏来抽象掉这个模式。

我们可以编写一个宏with-gensyms，它用来生成一些代码，而这些代码又用来生成另一些代码。宏with-gensyms应该类似下面这种形式：
```
(defmacro do-primes ((var start end) &body body)
  (with-gensyms (ending-value-name)
    `(do ((,var (next-prime ,start) (next-prime (1+ ,var)))
	(,ending-value-name ,end))
       ((> ,var ,ending-value-name))
       ,@body)))
```
with-gensyms 需要展开成一个LET，它会把每一个命名的变量都绑定到一个生成符号上。
```
(defmacro with-gensyms ((&rest names) &body body)
  `(let ,(loop for n in names collect `(,n (gensym)))
     ,@body))
```
以下示例演示loop是如何工作的。
```
CL-USER> (loop for x in '(a b c) collect `(,x (gensym)))
((A (GENSYM)) (B (GENSYM)) (C (GENSYM)))
```
