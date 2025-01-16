---
title: Common Lisp实践：建立单元测试框架
date: 2025-01-16 18:44:23
updated:
categories: Lisp
tags: [Common Lisp, 单元测试]
---
**单元测试框架**是一种工具或库，用于帮助开发人员创建、组织、运行和报告代码的单元测试。  
**单元测试**是指对软件系统中最小的可测试单元（通常是一个函数或方法）进行的验证测试。
<!-- more -->
常见的单元测试框架包括：

- **Python**: `unittest`、`pytest`
- **Java**: `JUnit`
- **C++**: `Google Test (gtest)`
- **JavaScript**: `Jest`、`Mocha`
- **C#**: `NUnit`
### **单元测试框架的意义**

1. **保证代码质量**：
    - 提前发现潜在的错误。
    - 避免重复引入已修复的bug。
2. **简化调试和维护**：
    - 通过测试定位问题点。
    - 提升代码可维护性。
3. **加速开发**：
    - 通过自动化测试减少手动验证的工作量。
4. **增强信心**：
    - 确保修改代码不会破坏已有功能。
5. **推动良好开发习惯**：
    - 提高模块化和解耦性。

## 一 两个最初的尝试
假设我们要为内置的“+”函数编写测试，那么下面这些可能是合理的测试用例：
```
(= (+ 1 2) 3)
(= (+ 1 2 3) 6)
(= (+ -1 -3) -4)
```
当然不必挨个进行测试，可以写个函数来执行上述测试用例。
```
(defun test-+ ()
  (and
   (= (+ 1 2) 3)
   (= (+ 1 2 3) 6)
   (= (+ -1 -3) -4)))
```
无论何时，当想运行这组测试用例时，都可以调用test-+，一旦它返回T，就知道所有测试用例通过了。但返回NIL时，说明有测试用例失败了，但无法确定是哪一个。为了找出哪些用例不通过，我们可以这样修改`test-+`函数。
```
(defun test-+ ()
   (format t "~:[FAIL~;Pass~] ... ~a~%" (= (+ 1 2) 3) '(= (+ 1 2) 3))
   (format t "~:[FAIL~;Pass~] ... ~a~%" (= (+ 1 2 3) 6) '(= (+ 1 2 3) 6)) 
   (format t "~:[FAIL~;Pass~] ... ~a~%" (= (+ -1 -3) -4) '(= (+ -1 -3) -4))) 
```
现在每个测试用例都单独报告结果。format指令中 `~:[FAIL~;Pass~]`会在format的第一个格式实参为假时打印FAIL，其他情况下为Pass。执行test-+函数如下：
```
CL-USER> (test-+)
Pass ... (= (+ 1 2) 3)
Pass ... (= (+ 1 2 3) 6)
Pass ... (= (+ -1 -3) -4)
NIL
```
当然这个版本的test-+函数也存在一些问题，首先是对format的重复调用过于冗余，尤其后面的测试表达式也存在重复，这些都急需重构。

## 二 重构
我们真正需要的应该是像第一个`test-+`那样能返回单一的T或NIL值的高效函数，但同时又能像第二个版本那样单独报告错误。

我们可以创建一个新函数来消除重复的format调用。
```
(defun report-result (result form)
  (format t "~:[FAIL~;Pass~] ... ~a~%" result form))
```
现在可以使用report-result来代替format。
```
(defun test-+ ()
  (report-result (= (+ 1 2) 3) '(= (+ 1 2) 3))
  (report-result (= (+ 1 2 3) 6) '(= (+ 1 2 3) 6))
  (report-result (= (+ -1 -3) -4) '(= (+ -1 -3) -4)))
```
接下来需要摆脱的是测试用例表达式的重复，理想的情况应该将表达式同时看作代码（为了获得结果）和数据（用来作为标签）。无论何时，若想将代码作为数据来看待，这就意味着需要一个宏。代码可能要写出下面这样：
```
(check (= (+ 1 2) 3))
```
并让其与下列形式等同。
```
(report-result (= (+ 1 2) 3) '(= (+ 1 2) 3))
```
很容易写出一个宏来做这种转换。
```
(defmacro check (form)
  `(report-result ,form ',form))
```
**`,form`**：插入表达式 `(= (+ 1 2) 3)` 的值（直接展开）。
**`',form`**：插入表达式 `(= (+ 1 2) 3)` 的符号表示形式，等价于 `'(= (+ 1 2) 3)`。
现在可以改变test-+来使用check。
```
(defun test-+ ()
  (check (= (+ 1 2) 3))
  (check (= (+ 1 2 3) 6))
  (check (= (+ -1 -3) -4)))
```
既然不喜欢重复的代码，为什么不把check的重复也一起消除掉。我们可以定义check来接受任意数量的形式并将它们中的每个都封装在一个对report-result的调用里。
```
(defmacro check (&body forms)
  `(progn
     ,@(loop for f in forms collect `(report-result ,f ',f))))
```
- **外层 `progn`**：
    
    - `progn` 是 Common Lisp 中的顺序执行结构。它依次执行其中的表达式，并返回最后一个表达式的值。
- **`loop` 循环**：
    
    - **`loop for f in forms`**：遍历 `forms` 中的每个表达式。
    - **`collect`**：将处理后的结果收集成一个列表。
    - **`(report-result ,f ',f)`**：对每个表达式 `f`，生成一个新的表达式：
        - `,f`：将 `f` 的内容作为求值部分插入。
        - `',f`：将 `f` 的原始符号形式插入为数据。
- **`,@` 的作用**：
    
    - **`,@`** 将 `loop` 返回的列表“解包”，作为 `progn` 的多个子表达式插入。

现在我们可以这样写出`test-+`。
```
(defun test-+ ()
  (check
    (= (+ 1 2) 3)
    (= (+ 1 2 3) 6)
    (= (+ -1 -3) -4)))
```

## 三 修复返回值
接下来修改`test-+`以使返回值可以指示所有测试用例是否都通过了。首先对`report-result`做点改变，让其可以在报告时顺便返回测试用例结果。
```
(defun report-result (result form)
  (format t "~:[FAIL~;Pass~] ... ~a~%" result form)
  result)
```
现在`report-result`返回了它的测试结果，但我们不能简单地使用`AND`,因为其存在短路，一旦某个测试用例失败就跳过了其余的测试。我们真正需要的是一个像`AND`那样的操作符，同时又没有短路行为。虽然Common Lisp不提供这样的构造，但我们通过宏可以轻松实现它。我们所需要的宏应该如下所示，现将其称为`combine-results`
```
(combine-results
  (foo)
  (bar)
  (baz))
```
并且上述代码应该与下面的代码等同。
```
(let ((result t))
  (unless (foo) (setf result nil))
  (unless (bar) (setf result nil))
  (unless (baz) (setf result nil))
  result)
```
编写这个宏的唯一麻烦之处，需要在展开式中引入一个变量，即前面代码中的`result`，在宏展开式中使用一个变量的字面名称会导致抽象层面出现漏洞，因此需要使用函数 GENSYM在其每次调用时返回唯一的符号，该符号在全局范围内唯一，避免用户代码中可能存在的变量名冲突。使用一个宏可以简化该过程。
```
(defmacro with-gensyms ((&rest names) &body body)
  `(let ,(loop for n in names collect `(,n (gensym)))
     ,@body))
```
通过使用`with-gensyms`，我们可以这样定义`combine-results`。
```
(defmacro combine-results (&body forms)
  (with-gensyms (result)
    `(let ((,result t))
       ,@(loop for f in forms collect `(unless ,f (setf ,result nil)))
       ,result)))
```
现在改变`check`来使用`combine-results`
```
(defmacro check (&body forms)
  `(combine-results
     ,@(loop for f in forms collect `(report-result ,f ',f))))
```
使用这个版本的`check`, `test-+`就可以输出它的三个测试表达式结果，并 返回T以说明所有测试用例都通过了。
```
CL-USER> (test-+)
Pass ... (= (+ 1 2) 3)
Pass ... (= (+ 1 2 3) 6)
Pass ... (= (+ -1 -3) -4)
T
```
如果改变一个测试用例而让其失败，最终的返回值也会变成NIL
```
CL-USER> (test-+)
Pass ... (= (+ 1 2) 3)
Pass ... (= (+ 1 2 3) 6)
FAIL ... (= (+ -1 -3) -5)
NIL
```

## 四 更好的结果输出
如果编写了大量测试，可能就需要以某种方式将它们组织起来，而不是将它们全部塞进一个函数里。例如，假设想要对“\*” 函数添加一些测试用例，可以写一个新测试函数。
```
(defun test-* ()
  (check
    (= (* 2 2) 4)
    (= (* 3 5) 15)))
```
现在有了两个测试函数，如果想用一个函数来运行所有测试用例，可以这样实现：
```
(defun test-arithmetic ()
  (combine-results
    (test-+)
    (test-*)))
```
为什么能复用`test-arithmetic`,我们通过`combine-results`的展开式就可以知道。
```
(LET ((#:G245 T))
  (UNLESS (TEST-+) (SETF #:G245 NIL))
  (UNLESS (TEST-*) (SETF #:G245 NIL))
  #:G245)
```
上述代码，当评估`(TEST-+)`的时候，就是评估了所有`(TEST-+)`下的测试用例。`test-arithmetic`的运行结果如下。
```
CL-USER> (test-arithmetic)
Pass ... (= (+ 1 2) 3)
Pass ... (= (+ 1 2 3) 6)
Pass ... (= (+ -1 -3) -4)
Pass ... (= (* 2 2) 4)
Pass ... (= (* 3 5) 15)
T
```
所有的用例都使用Pass来表示通过，如果能在测试结果中显示每个测试用例来自什么函数就好了，因为打印相关的代码使用`report-result`来表示，生成`report-result`的`check`并不知道它是从什么函数被调用的，这就意味着还需要改变调用`check`的方式，向其传递一个参数使其随后传递给`report-result`。

设计动态变量就是用于解决这类问题的。如果创建一个动态变量使得每个测试函数在调用`check`之前将其函数名绑定在其之上，那么`report-result`就可以无需理会`check`来使用它了。

第一步在最上层声明这个变量。
```
(defvar *test-name* nil)
```

对`report-result`稍微改动一些，使其在输出中包括`*test-name*`
```
(format t "~:[FAIL~;Pass~] ... ~a: ~a~%" result *test-name* form)
```

为了让`*test-name*`生效，还需要改变上述两个测试函数。
```
(defun test-+ ()
  (let ((*test-name* 'test-+))
    (check (= (+ 1 2) 3))
    (check (= (+ 1 2 3) 6))
    (check (= (+ -1 -3) -4))))

(defun test-* ()
  (let ((*test-name* 'test-*))
    (check
      (= (* 2 2) 4)
      (= (* 3 5) 15))))

```
现在结果都被正确打上了标签：
```
CL-USER> (test-arithmetic)
Pass ... TEST-+: (= (+ 1 2) 3)
Pass ... TEST-+: (= (+ 1 2 3) 6)
Pass ... TEST-+: (= (+ -1 -3) -4)
Pass ... TEST-*: (= (* 2 2) 4)
Pass ... TEST-*: (= (* 3 5) 15)
T
```

## 五 抽象诞生
现在已经实现的代码看似完整了，但还有值得改进的地方，比如对于测试函数`test-+`或者`test-*`，每个函数都需要包含其函数名两次，一次作为DEFUN中的名字，另一次在`*test-name*`的绑定里。导致重复是因为此时测试函数只做了一半抽象。为了得到一个完整抽象，需要用一种方法来表达“这是一个测试函数”，并且这种方法要能将所需的全部代码都生成出来。换句话说，你需要一个宏。

由于需要捕捉的模式是一个DEFUN加上一些样板代码，所以需要写一个宏使其展开成DEFUN，然后使用该宏去定义测试函数，可以将其命名为`deftest`。
```
(defmacro deftest (name parameters &body body)
  `(defun ,name ,parameters
     (let ((*test-name* ',name))
       ,@body)))
```
使用该宏，可以像下面这样重写`test-+`。
```
(deftest test-+ ()
  (check
    (= (+ 1 2) 3)
    (= (+ 1 2 3) 6)
    (= (+ -1 -3) -4)))
```

## 六 测试层次体系
如果想要将上千个测试用例组织在一起，建立合理的测试层次就很好理解。如果用`deftest`来定义诸如`test-arithmetic`这样的测试套件函数，并且对其中的`*test-name*`作一个小改变，就可以用测试用例的“全称”路径来报告结果，就像下面这样：
```
pass ... (TEST-ARITHMETIC TEST-+): (= (+ 1 2) 3)
```
因为已经抽象了测试函数的过程，所以就无需修改测试函数的代码从而改变相关的细节。为了使`*test-name*`保存一个测试函数名的列表而不只是最近进入的测试函数的名字，只需将绑定形式：
```
(let ((*test-name* ',name))
```
变成：
```
(let ((*test-name* (append *test-name* (list ',name))))
```
由于APPEND返回一个由其实参元素所构成的新列表，这个版本将把`*test-name*`绑定到一个含有追加其新名字到结尾处的`*test-name*`的旧内容的列表。每当一个测试函数返回时，`*test-name*`原有的值将被恢复。

现在可以用deftest代替DEFUN来重新定义test-arithmetic。
```
(deftest test-arithmetic ()
  (combine-results
    (test-+)
    (test-*)))
```
当然，`test-+`和`test-*`也应该用deftest定义。调用`test-arithmetic`结果如下。
```
CL-USER> (test-arithmetic)
Pass ... (TEST-ARITHMETIC TEST-+): (= (+ 1 2) 3)
Pass ... (TEST-ARITHMETIC TEST-+): (= (+ 1 2 3) 6)
Pass ... (TEST-ARITHMETIC TEST-+): (= (+ -1 -3) -4)
Pass ... (TEST-ARITHMETIC TEST-*): (= (* 2 2) 4)
Pass ... (TEST-ARITHMETIC TEST-*): (= (* 3 5) 15)
T
```
随着测试套件的增长，可以添加新的测试函数层次，只要用deftest来定义，结果就会正确输出。
```
(deftest test-math ()
  (test-arithmetic))
```
将如下输出。
```
CL-USER> (test-math)
Pass ... (TEST-MATH TEST-ARITHMETIC TEST-+): (= (+ 1 2) 3)
Pass ... (TEST-MATH TEST-ARITHMETIC TEST-+): (= (+ 1 2 3) 6)
Pass ... (TEST-MATH TEST-ARITHMETIC TEST-+): (= (+ -1 -3) -4)
Pass ... (TEST-MATH TEST-ARITHMETIC TEST-*): (= (* 2 2) 4)
Pass ... (TEST-MATH TEST-ARITHMETIC TEST-*): (= (* 3 5) 15)
T
```

## 九 总结
完整代码如下。

```
(defvar *test-name* nil)

(defmacro deftest (name parameters &body body)
  `(defun ,name ,parameters
     (let ((*test-name* (append *test-name* (list ',name))))
       ,@body)))
       
(defmacro with-gensyms ((&rest names) &body body)
  `(let ,(loop for n in names collect `(,n (gensym)))
     ,@body))

(defun report-result (result form)
  (format t "~:[FAIL~;Pass~] ... ~a: ~a~%" result *test-name* form)
  result)

(defmacro combine-results (&body forms)
  (with-gensyms (result)
    `(let ((,result t))
       ,@(loop for f in forms collect `(unless ,f (setf ,result nil)))
       ,result)))

(defmacro check (&body forms)
  `(combine-results
     ,@(loop for f in forms collect `(report-result ,f ',f))))
```

编写测试用例步骤：

对测试用例进行分类，例如"+"函数的合理分类为，test-+ << test-math <<  test-arithmetic
1. 先编写 `test-+`的测试用例
```
(deftest test-+ ()
  (check
    (= (+ 1 2) 3)
    (= (+ 1 2 3) 6)
    (= (+ -1 -3) -4)))

(deftest test-* ()
  (check
    (= (* 2 2) 4)
    (= (* 3 5) 15)))
```
2. 可选的，将 `test-+`放到一个更大的分类下，例如` test-arithmetic`
```
(deftest test-arithmetic ()
  (combine-results
    (test-+)
    (test-*)))
```
3. 可选的，将`test-arithmetic`放到一个更大的分类下，例如`test-math`
```
(deftest test-math ()
  (test-arithmetic))
```
4. 执行测试，执行最上层函数`test-math`就能执行所有的测试用例。