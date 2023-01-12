---
title: 用 Lisp 实现一个简单的数据库
date: 2021-11-16 11:12:14
updated:
categories: [Lisp]
tags: [Lisp,数据结构]
---
第一次认识 Lisp 是通过《黑客与画家》这本书，书中对 Lisp 赞不绝口，声称现在编程语言的发展也只是赶上了 1958 年的 Lisp 语言的水平。很多人就有疑问了，一个诞生于 1958 年的语言，计算机技术不是日新月异吗，为什么 Lisp 还没有过时？书中是这样说的，Lisp 是数学，数学是不会过时的。在书的作者 Paul Graham 的力荐下，对 Lisp 充满了浓厚的兴趣，就开始了 Lisp 的学习之旅。由于之前学习的繁忙，加上 Lisp 在"主流编程界" 好像并不受待见，我也没有一直持续学习，而是学习更加受欢迎的 C，Python 等语言，毕竟以后是要吃饭的嘛。在大学生涯的最后一段时间里，计算机基础知识学的越多，越发觉得编程语言的有趣之处，也明白了所有编程语言其实都是图灵等价的，即一个功能可以用任何编程语言实现，只不过是实现方式不一样。编程语言学的越多，越来越感受到 Lisp 本身设计的优雅，怀着一份好奇心，我又重新走进了 Lisp 的世界。
<!-- more -->
今天用一个例子来介绍 Lisp 的优雅之处。很多人可能有这样的想法，在用编程语言构建真实的软件之前，你必须先学会这门语言。现在，我们用 Lisp 中极少量的元素，来实现一个简单的数据库，用来存储 MP3 歌曲信息。

## CD 和记录

数据库包括多条 CD 记录，每条 CD 记录包括以下四个信息：CD 标题，艺术家信息，评价信息(满分 10分)，是否别烧录(布尔值)。

先介绍下等下需要用到的两种数据结构：

### 列表

称为 list，称为列表，类似于 Python 中的列表，例如：

以下环境类似于 Python 的交互模式：

```lisp
CL-USER> (list 1 2 3)
(1 2 3)
```

### 属性表(property list)

称为 plist

类似于 Python 中的字典，或 C++ 中的 map，由键值对组成，例如：

```lisp
CL-USER> (list :a 1 :b 2 :c 3)
(:A 1 :B 2 :C 3)
```

a，b，c 分别是是键，1，2，3 分别是值 

关于 Lisp 函数相关的介绍，不明白的小伙伴请参考之前的：[Common Lisp函数](https://wangjunstf.github.io/2021/04/26/common-lisp-han-shu/)

### 属性表查询

使用 getf 函数，例如：

```lisp
CL-USER> (getf (list :a 1 :b 2 :c 3) :a)
1
CL-USER> (getf (list :a 1 :b 2 :c 3) :b)
2
```

有了上述消息，就可以写出一个 make-cd 函数，它以参数的形式接受 4 个字段然后返回一个 plist

```lisp
(defun make-cd (title artist rating ripped)
  (list :title title :artist artist :rating rating :ripped ripped))
```

使用方法：

```lisp
CL-USER> (make-cd "Roses" "Kathy Mattea" 7 t)
(:TITLE "Roses" :ARTIST "Kathy Mattea" :RATING 7 :RIPPED T)
```

## 录入 CD

上述只算一个单一记录，还不算一个数据库，我们还需要一个更大的结构来保存记录。出于简化目的，以下使用一个全局变量 \*db\* ，名字中的星号是 Lisp 的全局变量名的命名约定。全局变量可以用 DEFVAR 宏来定义，不明白什么是宏也没关系，先暂时将它理解为函数。

```lisp
(defvar *db* nil)
```

使用 PUSH 宏为 \*db\*添加新的项，定义一个函数来给数据库增加一条记录：

```lisp
(defun add-record (cd) (push cd *db*))
```

现在可以将 add-record 和 make-cd 一起使用来为数据库添加新的 CD 记录。

```lisp
CL-USER> (add-record (make-cd "Roses" "Kathy Mattea" 7 t))

((:TITLE "Roses" :ARTIST "Kathy Mattea" :RATING 7 :RIPPED T) 3 13 123)
CL-USER> (add-record (make-cd "Pork Face" "Laddy" 9 t))
((:TITLE "Pork Face" :ARTIST "Laddy" :RATING 9 :RIPPED T)
 (:TITLE "Roses" :ARTIST "Kathy Mattea" :RATING 7 :RIPPED T) 3 13 123)
```

查看数据库的内容：

```lisp
CL-USER> *db*
((:TITLE "Pork Face" :ARTIST "Laddy" :RATING 9 :RIPPED T)
 (:TITLE "Roses" :ARTIST "Kathy Mattea" :RATING 7 :RIPPED T) 3 13 123)
```

这种输出方式可读性并不好，我们可以定义一个函数来格式化输出数据库信息：

lisp 中使用 format 函数来格式化输出字符串，它类似于 C 语言中的 printf

```lisp
(defun dump-db ()
  (dolist (cd *db*)
    (format t "~{~a:~10t~a~%~}~%" cd)))
```

dump-db 使用如下：注意不要使用 PUSH 往 \*db\*里存入其它不相干信息，不然调用 dump-db 会产生异常，因为它是按照add-record 返回的 plist 来解析的。如果存入了其它信息，就将  \*db\* 置空：`(set1 *db* nil)`

```lisp
CL-USER> (dump-db)
TITLE:    Pork Face
ARTIST:   Laddy
RATING:   9
RIPPED:   T

TITLE:    Roses
ARTIST:   Kathy Mattea
RATING:   7
RIPPED:   T

NIL
```

### DOLIST宏

类似于 C++ 中的 for-each 语法，用于遍历一个列表中的所有元素。

```lisp
(dolist (cd *db*)
    (format t "~{~a:~10t~a~%~}~%" cd))
```

依次将 \*db\* 中的每个元素绑定到 db 上，对于每个 cd 值，使用 format 函数打印它。

### FORMAT

```lisp
(format t "~{~a:~10t~a~%~}~%" cd)
```

虽然看起来有些晦涩，但其实非常简单和灵活。`~a` 表示一个占位符，类似于 C语言中的 `%d`，`~%` 表示换行，类似于 C语言中的 `\n`，`~10t`类似制表符，产生足够的空格，以确保在处理下一个`~a` 之前将光标移动 10 列。当 format 看到` ~{`，下一个被消耗的参数必须是一个列表，format 在列表上循环操作，处理位于 `~{`于`~}`之间的指令，同时在每次需要从列表上消耗掉尽可能多的元素。

从技术上讲，可以使用 format 在整个数据库本身上循环，从而将 dump-db 函数变为只有一行：

```lisp
(defun dump-db2 ()
    (format t "~{~{~a:~10t~a~%~}~%~}" *db*))
```

这究竟是可怕还是酷，完全取决于你的看法。



## 改进用户交互

使用 add-record 来添加 CD 记录显得太 Lisp 化了，如果想添加大量的记录，操作并不是很方便，可以写一个函数来提示提示用户录入任意条 CD 信息。

```lisp
(defun prompt-read (prompt)
  (format *query-io* "~a: " prompt)
  (force-output *query-io*)
  (read-line *query-io*))
```

 format 输出一个提示信息。

force-output 刷新缓冲区，有时提示信息没有及时输出，使用该 force-output 可以强制输出提示信息。

read-line 从标准输入读取一行内容。运行结果如下：

```lisp
CL-USER> (prompt-read "name")
name: stf

"stf"
NIL
```

现在可以将 prompt-read 和 make-cd 组合起来，从而构造一个可以根据提示输入每个值得到的数据中建立新的 CD 记录的函数。

```lisp
(defun prompt-for-cd ()
  (make-cd
   (prompt-read "Title")
   (prompt-read "Artist")
   (prompt-read "Rating")
   (prompt-read "Ripped [y/n]")))
```

上面的代码基本正确，但是 prompt-read 总是返回字符串，对于Title 和 Artist 来说可以，但对于 Rating 和 Ripped 就不太好了，它们应该是一个数字和一个布尔值。取决于你想要一个多专业的用户接口，花在验证用户输入的数据上的努力可以是无止境的。现在采取一个简单但不安全的方法，将 Rating 对应的 prompt-read 包装在一个 Lisp 的 PARSE-INEGER 函数里，就像这样：

```lisp
(parse-integer (prompt-read "Rating"))
```

这里存在的问题就是 parse-integer 的默认行为是当它无法从字符串中正确解析出整数或者字符串里含有任何非数字的垃圾值时直接报错。不过，它接受一个可选的关键字参数 :junk-allowed，可以让它适当地容忍一些，解析不出数字也不报错。

```lisp
(parse-integer (prompt-read "Rating") :junk-allowed t)
```

上面的代码的一个问题就是，当不能从垃圾中解析出数字时，parse-integer 返回 NIL，为了让这个思路更加完善，可以将 NIL 当作 0 来看待。这时可以使用 OR 宏，它与 C 中的逻辑运算符 `||`很类似，具有短路特性，OR 宏接受一系列表达式，依次求值它们，然后返回第一个非空值，如果全部是空值，就返回空值。

```
(OR (parse-integer (prompt-read "Rating") :junk-allowed t) 0)
```

修复 Ripped 的代码比较简单，只需使用 Common Lisp 的 Y-OR-N-P 函数

```
(y-or-n-p "Ripped [y/n]: ")
```

Y-OR-N-P 是相当健壮的，因为在没有得到 y,Y, n, N 时会提示重新输入。

综上所述，就能得到一个相当健壮的 prompt-for-cd 函数了。

```lisp
(defun prompt-for-cd ()
  (make-cd
   (prompt-read "Title")
   (prompt-read "Artist")
   (OR (parse-integer (prompt-read "Rating") :junk-allowed t) 0)
   (y-or-n-p "Ripped [y/n]: ")))
```

可以添加一个循环来一次性输入多个 CD 记录。

```lisp
(defun add-cds ()
  (loop (add-record (prompt-for-cd))
	(if (not (y-or-n-p "Another?[y/n]: ")) (return))))
```

使用如下：

```lisp
CL-USER> (add-cds)
Title: tom like rice
Artist: larry
Rating: 6

Ripped [y/n]:  (y or n) n

Another?[y/n]:  (y or n) y
Title: My sky
Artist: ck
Rating: 7

Ripped [y/n]:  (y or n) y

Another?[y/n]:  (y or n) n

NIL
CL-USER> *db*
((:TITLE "My sky" :ARTIST "ck" :RATING 7 :RIPPED T)
 (:TITLE "tom like rice" :ARTIST "larry" :RATING 6 :RIPPED NIL)
 (:TITLE "Pork Face" :ARTIST "Laddy" :RATING 9 :RIPPED T)
 (:TITLE "Roses" :ARTIST "Kathy Mattea" :RATING 7 :RIPPED T))
CL-USER> 
```

## 保存和加载数据库

以上代码可以方便地给数据库添加新记录，但是最大的缺陷就是重启 Lisp 后数据就消失了。幸运的是，借助我们用来表示数据的数据结构，可以相当容易地将数据保存在文件里以后重新加载。下面是一个 sava-db 函数，其接受一个文件名作为参数并且保存当前数据库的状态：

```lisp
(defun save-db (filename)
  (with-open-file (out filename
		       :direction :output
		       :if-exists :supersede)
    (with-standard-io-syntax
      (print *db* out))))
```

with-open-file 宏会打开一个文件，并将其绑定到一个变量，然后执行一组表达式，然后再关闭这个文件，它还可以保证即便在求值表达式出了错也可以正确关闭文件，紧跟着 with-open-file 的列表并非函数调用而是 with-open-file 的一部分，它包括用来在 with-open-file 主体中写入的文件流的变量名，一个必须是文件名的值，以及一些控制文件如何打开的选项，`:direction :output` 表示 打开一个用于写入的文件，`:if-exists :supersede`	表示如果存在同名文件就覆盖原来的文件。

一旦已经打开了文件，所需要做的就是使用 	`(print *db* out)` 将数据库的内容打印出来。跟 FORMAT 不同的是，PRINT 将 Lisp 对象打印成一种可以被 Lisp 读取器读回来的形式。宏 with-standard-io-syntax 可以确保影响 PRINT 行为的特定一些变量可以被设置为它们的标准值。当把数据读回来时，我们将使用同样的宏来确保 Lisp 读取器和打印器的操作彼此兼容。

save-db 的参数应该是一个含有用户打算用来保存数据库的文件名地址的字符串，字符串确切形式将取决于它们正在使用的操作系统，例如在 Unix 系统下可能会这样调用 save-db

```lisp
CL-USER> (save-db "/Users/wangjun/Desktop/java/my-cds.db")
((:TITLE "My sky" :ARTIST "ck" :RATING 7 :RIPPED T)
 (:TITLE "tom like rice" :ARTIST "larry" :RATING 6 :RIPPED NIL)
 (:TITLE "Pork Face" :ARTIST "Laddy" :RATING 9 :RIPPED T)
 (:TITLE "Roses" :ARTIST "Kathy Mattea" :RATING 7 :RIPPED T))
```

将数据加载回数据库的函数如下：

```lisp
(defun load-db (filename)
  (with-open-file (in filename)
    (with-standard-io-syntax
      (setf *db* (read in)))))
```

这次不要在 with-open-file 指定 `:direction` 了，应该这次使用的是默认的 `:input`，并且与打印相反，使用 READ 来从流中读入，这是与 REPL 所使用的相同的读取器，可以读取任何可以在 REPL 提示里输入的 Lisp 表达式。尽管如此，在这种情况下，它将只是读取和保存表达式，并不会对它求值。再一次，with-standard-io-syntax是为了 save-db 和 load-db 在 print 数据时相同的基本语法。

`setf` 宏是 Common Lisp 最主要的赋值操作符，它将其第一个参数设置成其第二个参数的求值结果，因此在 load-db 里 \*db\*变量将含有从文件中读取的对象，也就是由 save-db 所写入的那些列表的列表。值得注意的是：load-db 将改变  \*db\* 的值，如果已经往里添加了数据，但还没有调用 save-db 而直接调用 load-db 将丢失新添加的数据。

## 查询数据库

现在有了保存和重载数据的方法，以及一个便利的用户接口来添加新的记录，这样很快我们就会有足够多的记录以至于我们不想为了查看它有什么而每次都把整个数据库导出来。我们需要的是一种查询数据的方式，例如：

```lisp
(select :artist "Kathy Mattea")
```

然后就可以得到所有艺术家为 Kathy Mattea 的记录的列表。

函数 REMOVE-IF-NOT 接受一个谓词和一个列表，然后返回一个仅包含原来列表中匹配该谓词的所有元素所组成的新列表。谓词参数可以是任何接受单一参数并且返回布尔值的函数，除了 NIL 表示假以外，其余的都表示真。

例如，需要从一个数字组成的列表中解析出所有偶数，可以使用以下方法：

```lisp
CL-USER> (remove-if-not #'evenp '(1 2 3 4 5 6 7 8 9 10))
(2 4 6 8 10)
```

这里的谓词是函数 evenp，当其参数是偶数时返回真，那个有趣的 `#'`记号是**得到接下来的名字所对应的函数**的简称。如果没有 `#'`的话，Lisp 把 evenp 当作一个变量的名字来对待并查找该变量的值，而不是函数。`'(1 2 3 4 5 6 7 8 9 10)`中的 \`符号表示不对列表进行求值。

也可以给 REMOVE-IF-NOT 传递一个匿名函数。如果 EVENP 不存在，我们可以像下面这样来写前面的表达式。

```
CL-USER> (remove-if-not #'(lambda (x) (= 0 (mod x 2))) '(1 2 3 4 5 6 7 8 9 10))
(2 4 6 8 10)
```

在这种情况下，谓词是这样一个匿名函数：`(lambda (x) (= 0 (mod x 2)))` ，它会检查其参数模 2 等于 0(偶数)。如果想用匿名函数来解析出所有的奇数，可以这样写：

```
CL-USER> (remove-if-not #'(lambda (x) (= 1 (mod x 2)) ) '(1 2 3 4 5 6 7 8 9 10))
(1 3 5 7 9)
```

lambda 并不是函数名，它只是表面当前正在定义匿名函数。一个 lambda 表达式看起来很像一个 DEFUN ，单词 lambda 后面紧跟着形参列表，然后再是函数体。

为了用 remove-if-not 从数据库中挑选出所有 Kathy Mattea 的专辑，我们将需要一个可以在一条记录的艺术家字段是 "Kathy Mattea" 时返回真的函数。我们选择用 plist 来表达数据库里的记录，因为它可以使用 GETF 从 plist 解出给定名称的字段来，假设 cd 是数据库的一条记录，可以使用表达式：`(getf cd :artist)`解出艺术家的名字来。函数 EQUAL 当用于字符串参数时可以逐个字符地比较它们。因此 `(equal (getf cd :artist) "Kathy Mattea")` 用于测试一个给定 CD 的艺术家字段是否等于 "Kathy Mattea"。现在需要将这个表达式包装在一个 LAMBDA 形式里从而得到一个匿名函数传递给 REMOVE-IF-NOT，例如：

```
 (remove-if-not
 #'(lambda (cd) (equal (getf cd :artist) "Kathy Mattea")) *db*)
```

也可以将其包装为一个函数：

```
(defun select-by-artist (artist)
  (remove-if-not
 #'(lambda (cd) (equal (getf cd :artist) artist)) *db*))
```

除了根据 ARTIST 查询，我们还想根据 TITLE，RATING，RIPPED查询。我们可以写出一个更加通用的 select 函数，它接受一个函数作为其参数：

```
(defun select (selector-fn)
  (remove-if-not selector-fn *db*))
```

`#'`去哪了，这种情况下我们并不希望 remove-if-not 去使用一个名为 selector-fn 的函数，我们想要它使用的是一个作为 select 的参数传递到变量 select-fn 里的匿名函数。不过 #' 在调用 select 的时候需要。例如：

```
CL-USER> (select #'(lambda (cd) (equal (getf cd :artist) "ck")))
((:TITLE "My sky" :ARTIST "ck" :RATING 7 :RIPPED T))
```

这样显得对 select 的调用比较丑陋，幸运的是我们可以把匿名函数的创建过程包装起来：

```
(defun artist-selector (artist)
  #'(lambda (cd) (equal (getf cd :artist) artist)))
```

这是一个可以返回函数的函数，并且返回的函数里引用了一个在 artist-selector 返回以后将不会存在的变量——至少看起来是这样。使用方法：

```
CL-USER> (select (artist-selector "ck"))
((:TITLE "My sky" :ARTIST "ck" :RATING 7 :RIPPED T))
```

现在只需要更多的函数来生成选择器了，但是我们现在不想写 title-selector，rating-selector，ripped-selector。因为它们都具有相似的结构。为什么不写一个通用的选择器函数呢，一个根据传递给它的参数可以生成用于不同字段甚至其组合的选择器函数。现在先简要介绍一种称为关键字形参(keyword parameter)的语言特性。

以下是一个简单的形参列表：

```
(defun foo (a b c) (list a b c))
```

一个使用关键字形参的 foo 版本如下：

```
(defun foo (&key a b c) (list a b c))
```

调用方法如下：

```
CL-USER> (foo :a 1 :b 2 :c 3)
(1 2 3)
CL-USER> (foo :c 1 :b 2 :a 3)
(3 2 1)
CL-USER> (foo :c 1 :a 3)
(3 NIL 1)
CL-USER> 
```

正如示例所显示那样，变量 a、b和 c 的值被绑定到了跟在相应的关键字后面的值上，如果一个特定的关键字在调用时没有指定，那么对应的变量被设置为 NIL。有时可能需要区分作为参数显式传递给关键字形参的 NIL 和作为缺省值的 NIL。为了实现这一点，当指定一个关键字形参时，将那个简单的名称替换成一个包括参数名，缺省值和另一个称为 supplied-p 参数的列表。这个 supplied-p 被设置成真或假，具体取决于一个参数在特定的函数调用里是否真的作为关键字被传递了。下面是使用了该特性的 foo 版本：

```
(defun foo (&key a (b 20) (c 30 c-p)) (list a b c c-p))
```

使用方法如下：

```
CL-USER> (foo :a 1 :b 2 :c 3)
(1 2 3 T)
CL-USER> (foo :c 3 :b 2 :a 1)
(1 2 3 T)
CL-USER> (foo :a 1 :c 3)
(1 20 3 T)
CL-USER> (foo)
(NIL 20 30 NIL)
```

熟悉 SQL 就知道，我们使用 where 来进行查询，我们可以使用 where 来生成选择器函数。例如，像下面这样调用：

```
(select (where :artist "Kathy Mattea"))
```

或者：

```
(select (where :rating 10 :ripped nil))
```

该函数看起来像这样：

```lisp
(defun where (&key title artist rating (ripped nil ripped-p))
  #'(lambda (cd)
      (and
       (if title (equal (getf cd :title) title) t)
       (if artist (equal (getf cd :artist) artist) t)
       (if rating (equal (getf cd :rating) rating) t)
       (if ripped-p (equal (getf cd :ripped) ripped) t))))
```

这个函数返回一个匿名函数，后者返回一个逻辑 AND，其中每个子句来自于我们 CD 记录中的一个字段，每个子句会检查相应的参数是否被传递进来要么在参数，然后要么将其根 CD 记录中对应字段的值相比较，要么在参数值没有传进来时返回 t，也就是 Lisp 版本的逻辑真。**选择器函数将只在 CD 记录匹配所有传递给 where 的参数时才返回真**。注意使用 三元素列表来指定关键字形参 ripped，因为我们需要知道调用者是否实际传递了 `ripped nil`。

## 更新已有的记录——WHERE 的再次使用

现在已经有了完美通用化的 select 和 where 函数，是时候编写一个数据修改函数了，类似于 SQL 中的 update。我们现在已经有了 where 子句生成器。update 函数只是一些已经用过的思路的再次应用：使用一个通过参数传递的选择器函数来选取需要更新的记录，再使用关键字形参来指定需要改变的值。主要的新东西是对 MAPCAR 函数的使用，其映射在一个列表上——这里是 \*db\*，然后返回一个新的列表，其中包含在原来列表的每个元素上调用一个函数所得的结果。

```
(defun update (selector-fn &key title artist rating (ripped nil ripped-p))
  (setf *db*
	(mapcar
	 #'(lambda (row)
	     (when (funcall selector-fn row)
	       (if title (setf (getf row :title) title))
	       (if artist (setf (getf row :artist) artist))
	       (if rating (setf (getf row :rating) rating))
	       (if ripped-p (setf (getf row :ripped) ripped)))
	     row) *db*)))
```

使用方法：

```
CL-USER> (update (where :artist "ck") :rating 5)
((:TITLE "My sky" :ARTIST "ck" :RATING 5 :RIPPED T)
 (:TITLE "tom like rice" :ARTIST "larry" :RATING 6 :RIPPED NIL)
 (:TITLE "Pork Face" :ARTIST "Laddy" :RATING 9 :RIPPED T)
 (:TITLE "Roses" :ARTIST "Kathy Mattea" :RATING 7 :RIPPED T))
```

或者可以很容易地写一个函数来从数据库里删除记录：

```
(defun delete-rows (selector-fn)
  (setf *db* (remove-if selector-fn *db*)))
```

remove-if 和 remove-if-not 正相反，前者返回所有匹配元素都删除的列表。delete-rows 事实上改变了数据库的内容，只有 50多行代码，总共就这些。

所有代码如下：

```lisp
(defun make-cd (title artist rating ripped)
  (list :title title :artist artist :rating rating :ripped ripped))

(defvar *db* nil)

(defun add-record (cd) (push cd *db*))

(defun dump-db ()
  (dolist (cd *db*)
    (format t "~{~a:~10t~a~%~}~%" cd)))


(defun prompt-read (prompt)
  (format *query-io* "~a: " prompt)
  (force-output *query-io*)
  (read-line *query-io*))

(defun prompt-for-cd ()
  (make-cd
   (prompt-read "Title")
   (prompt-read "Artist")
   (OR (parse-integer (prompt-read "Rating") :junk-allowed t) 0)
   (y-or-n-p "Ripped [y/n]: ")))

(defun add-cds ()
  (loop (add-record (prompt-for-cd))
	(if (not (y-or-n-p "Another?[y/n]: ")) (return))))

(defun save-db (filename)
  (with-open-file (out filename
		       :direction :output
		       :if-exists :supersede)
    (with-standard-io-syntax
      (print *db* out))))


(defun load-db (filename)
  (with-open-file (in filename)
    (with-standard-io-syntax
      (setf *db* (read in)))))

(defun select (selector-fn)
  (remove-if-not selector-fn *db*))

(defun where (&key title artist rating (ripped nil ripped-p))
  #'(lambda (cd)
      (and
       (if title (equal (getf cd :title) title) t)
       (if artist (equal (getf cd :artist) artist) t)
       (if rating (equal (getf cd :rating) rating) t)
       (if ripped-p (equal (getf cd :ripped) ripped) t))))

(defun update (selector-fn &key title artist rating (ripped nil ripped-p))
  (setf *db*
	(mapcar
	 #'(lambda (row)
	     (when (funcall selector-fn row)
	       (if title (setf (getf row :title) title))
	       (if artist (setf (getf row :artist) artist))
	       (if rating (setf (getf row :rating) rating))
	       (if ripped-p (setf (getf row :ripped) ripped)))
	     row) *db*)))

(defun delete-rows (selector-fn)
  (setf *db* (remove-if selector-fn *db*)))
```

