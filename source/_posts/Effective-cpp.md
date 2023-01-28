---
title: Effective C++总结
date: 2023-01-12 17:31:47
updated:
categories: [编程语言]
tags: [C++,Effective C++]
---

## **导读**

学习程序语言根本大法是一回事；学习如何以某种语言设计并实现高效程序则是令一会事。

一组明智选择并精心设计的 classes，functions 和 templates 可使程序编写容易，直观，高效，并且远离错误。

### **1 explicit** 

default 构造函数：一个可被调用而不带任何实参的函数，这样的构造函数要不没有参数，要不就是每个参数都有缺省值。

<!-- more -->

```C%2B%2B
class A {
    public:
        A();          // default 构造函数
}

class B{
    explicit B(int x = 0, bool b = true);    // default 构造函数
}

class C{
    explicit C(int x);    //不是 default 构造函数
}
```

**explicit ：阻止类被用来执行隐式类型转换，但仍然可以进行显式类型转换。**

`C c = 123;  `         // 错误 C, explicit 禁止隐式类型转换

### **2 构造函数**

除非有一个好理由允许构造函数被用于隐式类型转换，否则就把它声明 explicit

```C%2B%2B
class Widget {
    public:
        Widget();                               // default 构造函数
        Widget(const Widget& rhs);              // copy 构造函数
        Widget& operator=(const Widget& rhs);   // copy assignment 操作符
}

Widget w1;                                      // 调用 default 构造函数
Widget w2(w1);                                  // 调用 copy 构造函数
w1 = w2;                                        // 调用 copy assignment 构造函数
```

**当看到等号时请小心，因为"="语法可用来调用 copy 构造函数**

```
Widget w3 = w2;                                 // 调用 copy 构造函数
```

**区分 copy 构造和 copy 赋值很容易，如果一个新对象被定义，例如上述  w3，一定会有一个构造函数被调用。如果没有新对象被定义，例如上面的 w1=w2，就不会有构造函数被调用，自然就是赋值操作被调用。**

### **3 避免对象的值传递**

copy 构造函数是一个尤其重要的函数，因为它定义了一个对象如何 passed by value（值传递）

```C%2B%2B
bool hasAcceptableQuality(Widget w);
...
Widget awidget;
if(hasAcceptableQuality(awidget))
```

上述参数 w 是以 by value 方式传递给 hasAcceptableQuality，所以在上述调用中 awidget 被复制到 w 体内。这个复制动作由 Widget 的 copy 构造函数完成。

**Pass-by-value**意味着调用 copy 构造函数，以 by value 传递用户自定义类型通常是个坏注意。pass-by-reference-to-const 往往是比较好的选择。例如：

```
bool hasAcceptableQuality(const Widget& w);
```

### **4 行为像函数的对象**

STL 中的许多相关机能以函数对象（function objects）实现，那是“行为像函数”的对象，这样的对象来自于重载 operator() （function call 操作符）的class。

### **5 未定义行为**

```C%2B%2B
int *p = 0;          // p 是一个 null 指针
std::cout << *p;     // 对一个 null 指针取值(dereferencing) 会导致未定义行为

char name[] = "Hello";           // name 是个数组，大小为 6（别忘记最末端的 null）
char c = name[10];               // 涉及一个无效的数组索引，会导致不明确行为
```

**未定义行为的结果不可预期，很可能让人不愉快，有时会耗费很多精力，因为它有时执行正常，有时造成崩溃，有时产出错误结果。**



## **一、让自己习惯 C++**

### **条款 01：视** **C++** **为一个语言联邦**

为了理解 C++ ，必须认识其主要的次语言，总共有 4 个：

1. C part of C++，当以 C++ 内的 C 充分工作时，高效编程守则映照出 C 语言的局限：没有模板，没有异常，没有重载
2. Object-Oriented C++：包括 classed（包括构造函数和析构函数），封装，继承，多态，虚函数（动态绑定）。这一部分是面向对象设计之古典守则在 C++ 上的最直接实施。
3. Template C++：C++ 的泛型编程部分，由于 templates 威力强大，它带来了崭新的编程范型（TMP，模板元编程）
4. STL：STL 是个 template，它对容器，迭代器，算法以及函数对象（function objects）的规约有极佳的紧密配合与协调。STL 有自己特殊的办事方式，当使用 STL 时，必须遵守它的规约。

对内置类型而言，pass-by-value 通常比 pass-by-reference 高效。

当从 C part of C++ 迁往 Object-Oriented C++ 时，由于用户自定义构造函数和析构函数的存在，pass-by-reference-to-const 往往更好。当运用 Template C++ 时尤其如此，因为你都不知道所处理的对象类型。

当进入 STL 之后，迭代器和函数对象都是在 C 指针之上塑造出来的，所以对 STL 的迭代器和函数对象而言，旧式的 pass-by-value 守则再次适用。

#### 总结

C++ 并不是一个带有一组守则的一体语言：它是从四个次语言组成的联邦政府，每个次语言都有自己的规约。记住这四个次语言就会发现 C++ 容易了解得多。

### **条款 02：尽量以 const, enum, inline 替换 #define**

#### 2.1 #define  定义常量缺点

可以用以下方式定义常量：

```
#define ASPECT_RATIO 1.653
```

> 记号名称 ASPECT_RATIO 也许从未被编译器看见，于是 ASPECT_RATIO 有可能没进入记号表（symbol table）内。当运用此常量但获得一个编译错误提示时，可能会带来困惑。错误提示中可能会提到 1.653 而不是 ASPECT_RATIO。如果 ASPECT_RATIO 被定义在其他人写的头文件中，也许你对 ASPECT_RATIO 来自何处毫无概念，于是将因为追踪它而浪费时间。

解决办法是以一个常量替换上述的宏：

```
const double AspectRation = 1.653;  // 大写名称通常用于宏
```

> 作为一个语言常量， AspectRation 肯定会被编译器看到，当然就会进入符号表内。此外对于浮点常量，使用常量可能比使用 `#define`导致较小量的码，因为预处理器“盲目将宏名称 ASPECT_RATIO 替换为 1.653 ”可能导致目标码(object code)出现多份 1.653，若改用常量 AspectRation 则不会出现这种情况。

当以常量替换 #define，有两种特殊情况值得说说：

#### 2.2 const 定义**常量指针**

常量定义式通常被放在头文件内（以便被不同的源码含入），因此有必要将指针（而不是指针所指之物）声明为 const。若要在头文件内定义常量的（不变的）字符串，需要写两次 const 。如下所示：

```
const char* const authorName = "Scott Meyers";
```

使用 string 对象通常比上面的方式合宜：

```
const std::string authorName("Scott Meyers");
```

#### 2.3 class 的专属常量

将常量的作用域限制在 class 内，则让它成为一个成员，

确保此常量至多只有一份实体，则让他成为一个 static 成员：

```C%2B%2B
class GamePlayer {
private:
    static const int NumTurns = 5;     // 常量声明式
    int scores[NumTurns];              // 使用该常量
}
```

然而上述的  NumTurns 只是声明式而非定义式。通常 C++ 要求你对你所使用的任何东西提供一个定义式，**但如果它是个 class 专属常量又是 static 且为整数类型（integral type，例如 ints, chars, bools）则需要特殊处理。只要不取它们的地址，就可以声明并使用它们而无需提供定义式。**

如果需要取某个 class 专属常量的地址，或不取地址但编译器（不正确地）要看到一个定义式，可在实现文件中定义如下：

```
const int GamePlayer::NumTurns;     // NumTurns 的定义式
```

class 常量已经在声明时获得初始值，因此定义时不可再设初值。

旧式的编译器也许不支持上述语法，它们不允许 static 成员在其声明式上获得初值，此外所谓的“in-class 初值设定”，也只允许对整数常量进行。如果编译器不支持上述语法，可以将初始值放在定义式：

```C%2B%2B
class CostEstimate{
private:
    static const double FudgeFactor;    // static class 常量声明
    ...                                 // 位于头文件内
};
```



```
const double CostEstimate::FudgeFactor = 1.35;     // static class 常量定义，位于实现文件内
```

#### 2.4 enum hack

当在 class 编译期间需要一个 class 常量值，万一编译器（错误地）不允许 "static 整数型class常量"完成"in class 初值设定"，可改用所谓的 “the enum hack” 补偿做法。其理论基础是：“一个属于枚举类型的数值可权充 int 被使用”，于是 GamePlayer 可被定义如下：

```C%2B%2B
class GamePlayer {
private:
    enum {NumTurns = 5};               // "the enum hack" —— 令 NumTurns 成为 5 的一个记号
    int scores[NumTurns];              // 这就没问题
}
```

enum hack 值得我们认识的数个理由：

1. enum hack 的行为某方面比较像 #define 而不像 const。例如取一个 const 的地址是合法的，但取一个 enum 的地址就不合法，而取一个 #define 的地址通常也不合法
2. 基于实用主义：许多代码用了它，所以看到它时你必须认识它。事实上 "enum hack" 是模板元编程的技术基础（见条款 48）



#### 2.5 #define  定义宏的缺点

另一个常见的 #define 误用情况是以它实现宏。宏看起来像函数，但不会造成函数调用。下面这个宏夹带着宏实参，调用函数 f

```C%2B%2B
// 以 a 和 b 的较大值调用 f
#define CALL_WITH_MAX(a,b) f( (a)>(b)?(a):(b) )
```

当以如下方式调用宏时：

```C%2B%2B
int a=5,b=0;
CALL_WITH_MAX(++a,b);           // a 被累加二次
CALL_WITH_MAX(++a,b+10);        // a 被累加一次
```

幸运的是你不需要对这种无聊事情提供温床，你可以获得宏带来的效率以及以一般函数的所有可预料行为和类型安全性（type safety），只需写出 template inline 函数。如下所示：

```C%2B%2B
template<typename T>
inline void callWithMax(const T& a, const T& b){
    f(a>b?a:b);
}
```

这个 inline 产出一群函数，每个函数都接受两个同型对象，并以其中较大者调用 f。这里不需要在函数体中为参数加上括号，也不需要操心参数被求值多次。此外由于 callWithMax 是个真正的函数，它遵守作用域（scope）和访问规则。例如你绝对可以写出一个 "class 内的 private inline 函数"。一般而言，宏无法完成此事。

#### 2.6 总结

对于常量，最好用 const 对象或 enum 对象替换 #define

对于形似函数的宏，最好改用 inline 函数替换 #define

### **条款 03：尽可能使用 const**

```
const int* const number = 1024;
```

如果关键字出现在 “*” 左边，表示被指物是常量，如果出现在“*” 右边，表示指针自身是常量。如果出现在 “*” 两边，表示被指物和指针两者都是常量。 

#### 3.1 迭代器与指针

STL 迭代器基于指针而设计，迭代器的作用像个 T* 指针。

- 声明迭代器为 const 就像声明指针为 const —— T* const，表示迭代器不得指向不同的东西
- 如果希望迭代器所指的东西不可被改动（模拟 const T*） ，使用 const_iterator

```C%2B%2B
std::vector<int> vec;
const std::vector<int>::iterator iter = vec.begin();          // iter 的作用像个 T* cosnt
*iter = 10;                                                   // 没问题，改变 iter 所指物
++iter;                                                       // 错误，iter 是 const
std::vector<int>::const_iterator cIter = vec.begin();         // cIter 是 const
*cIter = 10;                                                  // 错误,*cIter 是 const
++cIter;                                                      // 没问题，改变 cIter
```

####  3.2 函数返回常量值

```C%2B%2B
class Rational { ... }
const Rational operator* (const Rational& rhs);

Rational a,b,c;
```

令返回值为 const，可以避免以下问题，许多程序员无意那么做：

```
(a * b) = c;                   // 在 a * b 的成果上调用 operator=
```

const 可以预防那个 “没意思的赋值动作”

#### 3.3 const 成员函数

目的：使该成员函数可作用于 const 对象身上，const 对象只能调用 const 成员函数：

1. 它们使 class 接口比较容易被理解，得知哪个函数可以改动对象，哪个函数不行
2. 它们使 “操作 const 对象” 成为可能。

改善 C++ 程序效率的一个根本方法是以 pass by reference-to-const 方式传递对象，前提是我们有 const 成员函数用来处理 const 对象。

**两个成员函数如果只是常量性不同，可以被重载**

```C%2B%2B
class TextBlock {
public:
    const char& operator[](std::size_t position) const {return text[position]} //operator[] for const 对象
    char& operator[](std::size_t position) {return text[position];}  //operator[] for non-const 对象      
private:
    std::string text;
}

TextBlock tb("Hello");
std::cout << tb[0];          // 调用 non-const TextBlock::operator[]

const TextBlock ctb("World");
std::cout << ctb[0];         // 调用 const TextBlock::operator[]
只要重载 operator[] 并对不同的版本给予不同的返回类型，就可以令 const 和 non-const TextBlocks 获得不同的处理：
std::cout << tb[0];   // 没问题，读一个 non-const TextBlock
tb[0] = 'x';          // 没问题，写一个 non-const TextBlock
std::cout << ctb[0];  // 没问题，读一个 const TextBlock
ctb[0] = 'x';         // 错误，写一个 const TextBlock
```

const 成员函数不可以更改对象内任何 non-static 成员变量。

#### 3.4 物理常量性和逻辑常量性

bitwise constness（又称为 physical constness）和 logical constness。

- bitwise constness

成员函数只有在不更改对象之任何成员变量（static 除外）时才可以说是 const 。bitwise constness 正是 C++ 对常量性（constness）的定义，因此 const 成员函数不可以更改对象内任何 non-staic 成员变量。

不幸的是许多成员函数虽然不十足具备 const 性质却能通过 bitwish 测试，例如只有指针（而非其所指物）隶属于对象，即使只更改了“指针所指物”，那么称此函数为 bitwise const 不会引发编译器异议。

```C%2B%2B
class CTextBlock {
public:
    char &operator[](std::size_t position) const        // bitwise const 声明，但其实不恰当
    {return pText[position];}  
    
    private:
        char* pText;
}
```

operator[] 实现代码并不更改 pText，看看以下代码发生什么？

```C%2B%2B
const CTextBlock cctb("Hello");           // 声明一个常量对象
char* pc = &cctb[0];                      // 调用 const operator[] 取得一个指针，指向 cctb 数据
*pc = 'J';                                // cctb 现在变为 'Jello'
```

- logical constness

逻辑常量性：逻辑上维持常量性，但对象的某些 bits 可能被修改。

一个 const 成员函数可以修改它所处理的对象内的某些 bits。

```C%2B%2B
class CTextBlock {
public:
    std::size_t textLength;              // 最近一次计算的文本区块长度
private:
    char *pText;
    std::size_t textLength;              // 最近一次计算的文本区块长度
    bool lengthIsValid;                  // 目前长度是否有效
}

std::size_t CTextBlock::length const{
    if(!lengthIsValid) {
        textLength = std::strlen(pText);  // 错误，在 const 成员函数内
        lengthIsValid = true;             // 不能赋值给 textLength 和 lengthIsValid
        return textLength;
    }
}
```

上述实现当然不是 bitwise，因为 textLength 和 lengthIsValid 都可能被修改。那怎样才能通过编译器编译呢？

利用 C++ 的一个与 const 的摆动场：mutable（可变的）。mutable 释放掉 non-static 成员变量的 bitwish constness 约束。如下所示：

```C%2B%2B
class CTextBlock {
public:
    std::size_t textLength;              // 最近一次计算的文本区块长度
private:
    char *pText;
    mutable std::size_t textLength;              // 最近一次计算的文本区块长度
    mutable bool lengthIsValid;                  // 目前长度是否有效
}

std::size_t CTextBlock::length const{
    if(!lengthIsValid) {
        textLength = std::strlen(pText);  // 错误，在 const 成员函数内
        lengthIsValid = true;             // 不能赋值给 textLength 和 lengthIsValid
        return textLength;
    }
}
```

#### 3.5 在 const 和 non-const 成员函数中避免重复

```C%2B%2B
class TextBlock {
public:
    const char& operator[] (std::size_t position) cosnt{
        ...          // 边界检验
        ...          // 标记数据访问
        ...          // 校验数据完整性
        return text[position];
    }
    
    char& operator[](std::size_t position){
        ...          // 边界检验
        ...          // 标记数据访问
        ...          // 校验数据完整性
        return text[position];
    }
}
```

可以看到上述代码为了实现两种方式的返回值，有大量的代码冗余，可以使用下面的方式消除代码冗余：

```C%2B%2B
class TextBlock {
public:
    const char& operator[] (std::size_t position) const{
        ...          // 边界检验
        ...          // 标记数据访问
        ...          // 校验数据完整性
        return text[position];
    }
    
    // 首先将当前对象加上 const 调用 const 成员函数，然后将返回值中的 const 去掉
    char& operator[](std::size_t position){
        return 
            const_cast<char&> (                             // 将 op[] 返回值中的 const 转除
                static_cast<const TextBlock&>(*this)        // 为 *this 加上 const
                    [positon]                               // 调用 const op[]
            );   
    }
}
```

#### 3.6 总结

const 可以用于以下方面：在指针和迭代器身上；在指针，迭代器及references 设及的对象身上；在函数参数和返回类型身上；在 local 变量身上；在成员函数身上。

- 将某些东西声明为 const 可帮助编译器侦测出错误用法。const 可被施加于任何作用域内的对象，函数参数，函数返回类型，成员函数本体。
- 编译器强制实施 bitwish constness。但编写程序时应该使用“概念上的常量性（conceptual constness）”。
- 当 const 和 non-const 成员函数有着实质的的等价的实现时，令 non-const 版本调用 const 版本可避免代码重复。

### **条款04：确定对象被使用前已被初始化**

`int x;` x 此时并未被初始化，未初始化之前使用 会导致以下问题：

- 未定义行为
- 可能导致崩溃（某些平台）
- 读入一些半随即 "bits"，污染了正在进行读取动作的对象，导致不可预知的程序行为以及一些不愉快的调试过程。

**永远在使用对象之前先将它初始化。对于无任何成员的内置类型，必须手工完成此事。**

```C%2B%2B
int x = 0;                  // 对 int 进行手工初始化
const char* text = "A C-style string";        // 对指针进行初始化
double d;
std::cin >> d;                                // 以读取 input stream 的方式完成初始化
```

至于内置类型以外的任何其他东西，初始化责任落在构造函数。规则很简单，确保每一个构造函数都将对象的每一个成员初始化。

#### 4.1 成员初始化列表

不要混淆了赋值（assignment）和初始化（initialization）。例如下列代码：

```C%2B%2B
class PhoneNumber { ... };
class ABEntry{           // ABEntry = "Address Book Entry"
public:
    ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones);
private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber>thePhones;
    int numTimesConsulted;
}
ABEntry::ABEtry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones){
    // 以下都是赋值，而非初始化
    theName = name;   
    theAddress = address;
    thePhones = phones;
    numTimesConsulted = 0;
}
```

这不是最佳做法，C++ 规定，对象的成员函数的初始化动作发生在进入构造函数本体之前。ABEntry 构造函数的一个较佳写法，使用所谓的 member initialzation list（成员初始化列表）：

```C%2B%2B
ABEntry::ABEtry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones):theName(name),theAddress(address),thePhones(phones),numTimesConsulted(0){
    // 以下都是赋值，而非初始化
}
```

上述效率通常更高。基于赋值的那个版本首先调用 default 构造函数为 theName, theAddress 和 thePhones 设初值，然后立刻对它们赋予新值，default 构造函数的一切作用因此浪费了。初始值列表避免了这一问题。**因为初值列中各个成员变量而设的实参，被拿去作为各成员变量之构造函数的实参。**本例中的 theName 以 name 为初值进行 copy 构造，theAddress 以 address 为初值进行 copy 构造，thePhones 以 phones 为初值进行 copy 构造。

**对大多数类型，比起先调用 default 构造函数，再调用 copy assignment（拷贝赋值运算符）。单只调用一次 copy 构造函数是比较高效的 。**对于内置对象 numTimesConsulted，其初始化和赋值的成本相同，但为了一致性最好也通过成员初值列表来进行初始化。

或者当想要 default 构造一个成员变量，都可以使用初始值列表。

```C%2B%2B
ABEntry::ABEntry()
:theName(),                        // 调用 theName 的 default 构造函数
theAddress(),                      // 同上
thePhones(),                       // 同上
numTimesConsulted(0)               // 记得将  numTimesConsulted 显式初始化为0
{} 
```

规则：规定总是在初始值列中列出所有成员变量，以免还得记住哪些成员变量可以无需初值。例如：如果 numTimesConsulted 属于内置类型，但初始值列表遗漏了它，它就可能会获得随即值。

如果成员变量是 const 或 references ，它们就一定需要初值，不能被赋值。

C++ 的“成员初始化次序”，先初始化基类，然后初始化派生类。class 的成员变量总是以其声明次序被初始化，和成员初始化列表的次序没有关系。

#### 4.2 non-local static 对象

static 对象，其寿命从被构造出来直到程序结束为止。因此 stack 和 heap-based对象被排除。static 对象包括 global 对象，定义于 namespace 作用域内的对象，在 classes 内，在函数内，以及在 file 作用域内被声明为 static 的对象。函数内的 static 对象称为 local static 对象（因为它们对函数而言是 local）。

其他 static 对象称为 non-local static 对象。程序结束时 static 对象会被自动销毁，也就是它们的析构函数会在 main() 结束时自动调用。

编译单元：产出单一目标文件的那些源码。

某个编译单元的某个 non-local static 对象的初始化动作使用了另一个编译单元的 non-local static ，它所用的对象可能尚未被初始化，因为 C++ 对 "定义于不同编译单元内的 non-local static 对象"的初始化次序并无明确规定。以下举例说明：

假设现有一个 FileSystem class ，它让互联网上的文件看起来好像位于本机（local）。你可能会产出一个特殊对象，位于 global 或 namespace 作用域内，象征单一文件系统：

```C%2B%2B
class FileSystem{
public:
    std::size_t numDisks() const;                  // 众多成员函数之一
};
```

注意 tfs 对象，如果在它未构造完成之前就使用它，会得到惨重的灾情。 



现在假设某些客户建立了一个 class 用以处理文件系统内的目录（directories），很自然他们的 class 会用上 tfs 对象：

```C%2B%2B
extern FileSystem tfs;                             // 预备给客户使用的对象

class Directory{            // 由程序库客户建立
public:
    Directory(params);
    ...
};

Directory:: Directory(params){
    ...
    std::size_t disks = tfs.numDisks();   // 使用 tfs 对象
    
}
```

进一部假设，这些客户决定创建一个 Directory 对象，用来放置临时文件：

```C%2B%2B
Directory tempDir(params);                   // 为临时文件而做出的目录
```

除非  tfs 在 temDir 之前初始化，否则 temDir 的构造函数会用到尚未初始化的 tfs。但 tfs 和 temDir 是不同的人在不同的时间于不同的源码文件中建立起来的，它们是定义于不同编译单元内的 non-local static 对象。

幸运的是一个小小的设计便可以完全消除这个问题 。唯一需要做的就是：

将每个 non-local static 对象搬到自己的专属函数内（该对象在此函数内被声明为 static）。这些函数返回一个 reference 指向它所含的对象。然后用户调用这些函数，而不直接指涉这些对象。换句话说，non-local static 对象被 local static 对象替换了。

从 Design Pattern 上来说，这属于 Singleton 模式的一个常见实现手法。

这个手法的基础在于：C++ 保证，函数内的 local static 对象会在 "该函数被调用期间"  "首次遇上该对象之定义式" 时被初始化。以 "函数调用" 替换 "直接访问 non-local static 对象" 就保证了所获得的那个 reference 将指向一个经历初始化的对象。更棒的是，如果从未调用 non-local static 对象的"仿真函数"，就绝不会引发构造和析构成本。

```C%2B%2B
class FileSystem {...};                    // 同前
FileSystem& tfs(){                        // 这个函数替换 tfs 对象
    static FileSystem fs;
    return fs;                            // 返回一个 reference 指向上述对象
}
class Directory { ... };                  // 同前
Directory::Directory(params){  // 同前，但原本的 reference to tfs，现在改为 tfs()
    ... 
    std::size_t disks = tfs().numDisks();
    ...
}              

Directory& tempDir(){                   // 这个函数用来替换 tempDir 对象
    static Directory td;                // 它在 Directory class 中可能是个 static
    return td;                          // 返回一个 reference 指向上述对象
}
```

缺点：这些函数内含 static 对象的事实使它们在多线程系统中带有不确定性。任何一种 non-const static 对象，无论它是 local 或 non-local，在多线程下"等待某事发生"都会有麻烦。

解决办法：在程序的单线程启动阶段手工调用所有 reference-returning 函数，这可消除与初始化有关的"竞速形势"。

**为避免对象初始化之前过早地使用它们，你需要做三件事。第一：手工初始化内置 non-member 对象。第二，使用成员初始化列表对付对象的所有成分。最后在"初始化次序不确定性"氛围下加强你的设计。**

#### 4.3 总结

- 为内置类型进行手工初始化，因为 C++ 不保证初始化它们
- 构造函数最好使用成员初始化列表，而不要在构造函数本体内使用赋值操作。
- 为免除 “跨编译单元之初始化次序”问题，请以 local static 对象替换 non-local static 对象。

## 二、构造/析构/赋值运算

每个 class 都会有一或多个构造函数，一个析构函数，一个 copy assignment 操作符。得确保它们的行为正确，因为它们是 class 的脊柱。

### 条款05：了解 C++ 默默编写并调用哪些函数

一个类即使没有声明任何函数，编译器也会为它声明一个 copy 构造函数，一个 copy assignment 操作符和一个析构函数，一个 default 构造函数。所有这些函数都是 public 且 inline。

```C%2B%2B
class Empty {};
```

经过编译器，它会变成这样：

```C%2B%2B
class Empty {
    Empty() { ... }                   // default 构造函数
    Empty(const Empty& rhs) { ... }   // copy 构造函数
    ~Empty() { ... }                  // 析构函数，是否是 virtual 见稍后说明
    
    Empty& operator=(const Empty& rhs) { ... } // copy assignment 操作符
};
```

只有当这些函数被需要（被调用），它们才会被编译器创建出来。程序中需要它们是很平常的事：

```C%2B%2B
{  
    Empty e1;             // default 构造函数             
    Empty e2(e1);         // copy 构造函数
    e2 = e1;              // copy assignment
                          // 析构函数                         
}
```

编译器为啥要生成这些函数呢？default 构造函数和析构函数主要是给编译器一个地方用来放置 "藏身幕后"的代码，像是调用 base classes 和 non-static 成员变量的构造函数和析构函数。**注意：编译器产出的析构函数是个 non-virtual 。除非这个 class 的 base class 自身声明有 virtual 析构函数（这种情况下这个函数的虚属性主要来自 base class）**

至于 copy 构造函数和 copy assignment 操作符，编译器创建的版本只是单纯地将来源对象的每一个 non-static 成员变量拷贝到目标对象。考虑一个 NameObject template，它允许你将一个个名称和类型为 T 的对象产生关联：

```C%2B%2B
template<typename T>
class NamedObject{
public:
    NamedObject(const char* name, const T& value);
    NamedObject(const std::string& name, const T& value);
    ...
private:
    std::string nameValue;
    T objectValue;
}
```

由于其中已经声明了一个构造函数，编译器就不再为它创建一个 default 构造函数，但会为它创建 copy 构造函数和 copy assignment 操作符。copy 构造函数的用法如下：

```C%2B%2B
NamedObject<int> no1("Smallest Prime Number",2);
NamedObject<int> no2(no1);                          // 调用 copy 构造函数
```

编译器生成的 copy 构造函数必须以 no1.nameValue 和 no1.objectValue 为初值设定 no2.namevalue 和 no2.objectValue。no2.nameValue 的初始化方式是调用 string 的 copy 构造函数。另一个成员 NameObject<int>::objectValue 的类型是 int，因此 T 是 int，是个内置类型，所以 no2.objectValue 会以拷贝 no1.objectValue 内的每一个 bits 来完成初始化。



编译器为 NamedObject<int> 所产生的 copy assignment 操作符。一般而言只有当产生的代码合法且有适当机会证明它有意义，万一两个条件有一个不符合，编译器会拒绝 class 产出 operator=

举例，NamedObject 定义如下，其中 nameValue 是个 reference to string, objectValue 是个 const T：

```C%2B%2B
template<typename T>
class NamedObject{
public:
    // 以下构造函数不再接受一个 const 名称，因为 nameValue 是个 reference-to-non-const string
    NamedObject(const std::string& name, const T& value);
    ...
private:
    std::string& nameValue;           // 现在是一个 reference
    const T objectValue;              // 现在是个 const 
};
```

现在考虑以下情况：

```C%2B%2B
std::string newDog("PersePhone");
std::string oldDog("Satch");
NameObject<int> p(newDog,2);           // 我们的狗 PersePhone 即将度过其第二个生日
NameObject<int> s(oldDog,36);          // Satch 现在 36 岁
p = s;                                 // 现在 p 成员变量发生了什么     
```

该赋值动作如何影响 p.nameValue 呢？难道 p.nameValue 应该指向 s.nameValue吗？当然是不是的，因为 C++ 并不允许“让 reference 改指向不同对象”。或者 p.nameValue 所指的哪个 string 该被修改，进而影响持有 pointers 或 references 而且指向该 string 的其他对象吗？

- 面对这个难题，C++ 的响应是拒绝编译那一行赋值动作。如果你打算在一个“内含 reference 成员”的 class 内支持赋值操作，就必须定义自己的 copy assignment 操作符。
- 面对 “内含 const 成员”，编译器的反应也一样。
- 还有一种情况，如果某个 base classes 将 copy assignment 操作符声明为 private，编译器将拒绝为其 derived classes 生成 copy assignment。

#### 总结 

编译器可以暗自为 class 创建 default 构造函数，copy 构造函数，copy assignment 操作符，以及析构函数。

### 条款 06：若不想使用编译器自动生成的函数，就应该明确拒绝

#### 6.1 不安全的做法：

所有编译器产生的函数都是 public ，为阻止这些函数被创建，得自行声明他们，可以将 copy 构造函数或 copy assignment 操作符声明为 private。

一般而言这个做法并不绝对安全，因为 member 函数和 friend 函数还是可以调用 private 函数，除非不去定义他们。

#### 6.2 安全做法

将 copy 构造函数或 copy assignment 操作符声明为 private，且故意不实现他们。这样当客户企图拷贝某个对象，编译器会阻饶它。如果不慎在 member 函数或 friend 函数之内那么做，轮到连接器发出抱怨。

#### 6.3 更聪明的做法

将连接期错误移至编译器是可能的，而且是好事，毕竟越早发现错误越好。将 copy 构造函数或 copy assignment 操作符声明为 private，但不是在目标对象内，而是在一个专门为了阻止 copying 动作而设计的 base class 内：

```C%2B%2B
class Uncopyable {
protected:
    Uncopyable() {}                   // 允许 derived 对象构造和析构
    ~ Uncopyable() {}
private:
    Uncopyable(const Uncopyable&);    // 阻止 copying
    Uncopyable& operator=(const Uncopyable&);
}
```

为了阻止拷贝 HomeForSale 对象，唯一需要做的就是继承 Uncopyable

```C%2B%2B
class HomeForSale: private Uncopyable{         // class 不再声明 copy 构造函数和 copy assignment 操作符
    ...
};
```

这样的话，只要任何人——甚至是 member 函数或 friend 函数，尝试拷贝 HomeForSale 对象，编译器便尝试生成一个 copy 构造函数和一个 copy assignment 操作符号，而这些函数会尝试调用其 base class 的对应兄弟，那些调用会被编译器拒绝，因为其 base class 的拷贝函数是 private。

#### 6.4 总结

为驳回编译器自动生产 copy 构造函数和 copy assignment 操作符。可以将相应的成员函数声明为 private 并且不予实现。或者像 Uncopyable 这样的 base class 也是一种做法。

### 条款 07：为多态基类声明 virtual 析构函数

#### 7.1 virtual 析构函数的作用

当一个 derived class 对象经由一个 base class 指针被删除，而该 base class 含有一个 non-virtual 析构函数，其结果未定义。实际可能发生的是对象的 derived 成分没有被销毁，其 base class 成分通常会被销毁。这就造成了诡异的 “局部销毁”对象。

消除上述问题的办法，给 base class 一个 virtual 析构函数

```C%2B%2B
class TimeKeeper {
public:
    TimeKeeper();
    virtual ~TimeKeeper();
};

TimeKeeper* ptk = getTimeKeeper();
...
delete ptk; 
```

像 TimeKeeper 这样的 base classes 除了析构函数之外通常还有其他 virtual 函数，因为 virtual 函数的目的就是允许 derived class 的某些成员函数采用不同的源码实现，以实现多态。 

**任何 class 只要带有 virtua 函数都几乎确定应该也有一个 virtual 析构函数。**

#### 7.2 不含 virtual 函数的好处

如果 class 不含 virtual 函数，通常表示它不意图被用作一个 base class。当一个 class 不被企图当作一个 base class，令其析构函数为 virtual 往往是个搜主意。例如：

```C%2B%2B
class Point {
public:
    Poing(int x,int y);
    ~Point();
private:
    int x,int y;
};
```

如果 int 占 32 bits，那么 Point 对象可塞入一个 64-bit 缓冲器中。这样一个 Point 对象可被当作一个 “64-bit 量”传给以其他语言如 C 或 FORTRAN 写的函数。

然而当 Point 的析构函数是 virtual ，情况就大不相同了：

为了实现 virtual 函数，对象必须携带某些信息，主要用来在运行期决定哪一个 virtual 函数被调用，这份信息通常由一个所谓 vptr(virtual table Pointer)指针指出。vptr 指向一个由函数指针构成的数组，称为 vtbl(virtual table)，每一个带有 virtual 函数的 class 都带有一个相应的 vtbl。当对象调用某一 virtual 函数，实际被调用的函数取决于该对象的  vptr 所指的那个 vbtl——编译器在其中寻找适当的函数指针。

#### 7.3 virtual 函数的误用

##### 移植相关

1. 当一个 class 含有虚函数，其对象的体积会增加，在 32-bit 计算机体系结构中将占用 96 bits（两个 int 和一个 vptr）。在 64 bits 计算机体系结构中可能占用 128 bits（两个 int 和一个 64 bits 指针）。为 Point 添加一个 vptr 会增加其对象大小达 50%~100%，Point 对象将不再能够塞入一个 64-bit 缓冲器。因此也就不再可能把它传递至（或接受自）其他语言所写的函数（因为其他语言的对应物并没有 vptr），也因此不再具有移植性。

##### 继承相关

只有当 class 内含有至少一个 virtual 函数，才为它声明 virtual 析构函数。即使 class 完全不带 virtual 函数，造成“non-virtual 析构函数问题”也有可能。举个例子，标准 string 不含任何 virtual 函数，有时候程序员会错误地把它当作 base class：

```C%2B%2B
class SpecialString: public std::string {  // 搜主意： std::string 有个 non-virtual 析构函数
    ...
};
```

看似无害，如果无意间将一个 pointer-to-SpecialString 转换为一个 pointer-to-string，然后将转换所得的那个 string 指针 delete 掉。就会引发诡异的“局部销毁”对象：

```C%2B%2B
SpecialString *pss = new SpecialString("Test Dog");
std::string *ps;
...
ps = pss;
...
delete ps;               // 未定义，*ps 的 SpecialString 资源会泄漏，因为 SpecialString 的析构函数没有被调用
```

相同的分析适用于任何不带 virtual 析构函数的 class，包括所有的 STL 容器如 vector，list，set，trl::unordered_map。

**很不幸，C++ 没有提供类似 Java 的 final classed 或 C# 的 sealed class 那样的 “禁止派生机制”。**

#### 7.4纯虚函数

纯虚函数导致抽象 classes，也就是不能被实例化的 class。由于抽象 class 总是企图被当作一个 base class 来用，而又由于 base class 应该有个 virtual 析构函数，并且由于纯虚函数会导致抽象 class。还有一点，有时候希望拥有抽象 class ，但手上没有任何 pure virtual 函数，怎么办？

解法很简单，为你希望它成为抽象的那个 class 声明一个 pure virtual 析构函数，例如：

```C%2B%2B
class AWOV {
public:
    virtual ~AWOV() = 0;                 // 声明 pure virtual 析构函数
};
```

必须为这个 pure virtual 析构函数提供一份定义：

```C%2B%2B
AWOV::~AWOV() {}      // pure virtual 析构函数的定义
```

析构函数的运作方式是，最深层派生的那个 class 其析构函数最先被调用，然后是其中每一个 base class 的析构函数被调用。编译器会在 AWOV 的 derived classes 的析构函数中创建一个对 ~AWOV 的调用动作，所以你必须为这个函数提供一份定义。

“给 base classes 一个 virtual 析构函数”，这个规则只适用于带多态性质的 base classes 身上。这种 base classes 的设计目的是为了用来“通过 base class 接口处理 derived class 对象”。

并非所有 base classes 的设计目的都是为了多态用途。例如标准 string 和 STL容器都不被设计作为 base classes 使用，它们并非被设计用来“经由 base class 接口处理 derived class 对象”，因此它们不需要 virtual 析构函数。

#### 7.5 总结

Polymorphic（带多态性质的）base classes 应该声明一个 virtual 析构函数。如果 class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数

class 的设计目的不是作为 base classes 使用，或不是为了具备多态，就不该声明 virtual 析构函数。

### 条款 08：别让异常逃离析构函数

C++ 并不禁止析构函数吐出异常，但它不鼓励你这样做。在两个异常同时存在的情况下，程序若不是结束执行，就是导致不明确行为。

假设有一个 class 负责数据库连接：

```C%2B%2B
class DBConnection {
public:
    ...
    static DBConnection create();           // 返回 DBConnection 对象
    void close();                           // 关闭连接，失败则返回异常                                                                    };
```

为了确保不会忘记在 DBConnection 身上调用 close()，一个合理想法就是创建一个用来管理 DBConnection 资源的 class，并在其析构函数中调用 close：

```C%2B%2B
class DBCon {
public:
    ...
    ~DBCon(){            // 确保数据库连接总是会被关闭
        db.close();
    }
private:
    DBConnection db;
};
```

假设客户端写出以下代码：

```C%2B%2B
{
DBCon dbc(DBConnection::create()); // 建立 DBConnection 对象并交给 DBConn 对象管理
                                   // 通过 DBConn 的接口使用 DBConnection 对象
}                                  // 在区块结束点，DBConn 对象被销毁，因而自动为 DBConnection 对象调用 close 
```

只要调用 close 成功，一切都美好。如果调用导致异常，DBConn 析构函数会传播该异常，也就是允许它离开这个析构函数。那会造成问题。

有两个办法可以避免这个问题，DBConn 析构函数可以：

1. 如果 close 抛出异常就结束程序，通常通过调用 abort 完成：

```C%2B%2B
DBConn::~DBConn()
{
    try {db.close();}
    catch (...){
        制作运转记录，记下对 close 的调用失败
        std::abort();
    }
}
```

如果程序遭遇一个“于析构期间发生的错误”后无法继续执行，”强迫结束程序“是个合理选项。毕竟它可以阻止异常从析构函数中传播出去（那会导致不明确的行为）。

1. 吞下因调用 close 而发生的异常

```C%2B%2B
DBConn::~DBConn(){
    try{db.close();}
    catch(...){
        制作运转记录，记下对 close 的调用失败
    }
}
```

一般而言，将异常吞掉是个坏主意，因为它压制了“某些动作失败”的重要信息。然而有时候吞下异常也比负担“草率结束程序”或“不明确行为带来的风险”好。

这些办法都没有什么吸引力，问题在于两者都无法对“导致 close 抛出异常”的情况做法反应。

一个较佳策略是重新设计 DBConn 接口，使其客户有机会对可能出现的问题作出反映。例如 DBConn 自己可以提供一个 close 函数，因而赋予客户一个机会得以处理“因该操作而发生的异常”。DBConn 也可以追踪其所管理的 DBConnection 是否已经被关闭，并在答案为否的情况下由其析构函数关闭之。然而如果 DBConnction 析构函数调用 close 失败，我们又将退回到 “强迫程序结束”或“吞下异常”的老路。

```C%2B%2B
class DBCon {
public:
    ...
    void close(){        // 供客户使用的新函数
        db.close();
        closed = true;
    }
    
    ~DBCon(){            // 关闭连接，如果客户不那么做的话
        if(!closed){
            try {
                db.close();
            }
            catch(...){
                制作运转记录，记下对 close 的调用失败     // 如果关闭动作失败，记下并结束程序或吞下异常
            }
        }
    }
    
private:
    DBConnection db;
    bool closed;
};
```

如果某个操作可能在失败时抛出异常，而又存在某种需要必须处理该异常，那么这个异常必须来自析构函数以外的某个函数。因为析构函数吐出异常就是危险，总会带来“过早结束程序”或“发生不明确行为”的风险。

在本例中，由客户自己调用 close 并不会给他们带来负担，而是给它们一个处理错误的机会，他们也可以忽略它们，依赖 DBConn 析构函数去调用 close。

#### 总结

析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序，

如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么 class 应该提供一个普通函数（而非在析构函数中）执行该操作。



### 条款09：绝不在构造和析构过程中调用 virtual 函数

#### 9.1 构造函数中调用虚函数

假设现有一个模拟股市交易如买进，卖出的订单等等，每当创建一个交易对象，需要在审计日志中创建一笔记录，如下所示：

```C%2B%2B
class Transaction {                                // 所有交易的 base class
public:
    Transaction();
    virtual void logTransaction() const = 0 ;      // 做出一份因类型不同而不同的日志记录  
    ...
};

Transaction:: Transaction(){
    ...
    logTransaction();                    // 最后动作是记录这笔交易
}

class BuyTransaction: public Transaction {          // derived class
public:
    virtual void logTransaction() const;            // 标记此类型交易
}

class SellTransaction: public Transaction {          // derived class
public:
    virtual void logTransaction() const;            // 标记此类型交易
}
```

现在执行以下操作：

```C%2B%2B
BuyTransaction b;
```

首先肯定 base class 的构造函数先被执行，然后执行 derived class 构造函数。上述代码中在构造函数中调用了虚函数 logTransaction，虽然现在创建的是 BuyTransaction，但 base class 构造函数中调用的虚函数是 Transaction 的版本。

base class 构造期间 virtual 函数绝不会下降到 derived classes 阶层。在 base class 构造期间， virtual 函数不是 virtual 函数。

当 base class 构造函数执行时 derived class 的成员变量尚未初始化，如果此期间调用的 virtual 函数下降至 derived classes 阶层，要知道 derived classes 的函数几乎必然取用 local 成员变量，而那些变量尚未初始化。

在 derived class 对象的 base class 构造期间，对象的类型是 base class 而不是 derived class。

#### 9.2 析构函数中调用虚函数

 析构函数也是类似的。一旦 derived class 析构函数开始执行，对象内的 derived class 成员变量便呈现未定义值，所以 C++ 视它们彷佛不存在。进入 base class 析构函数后对象就成为一个 base class 对象

#### 9.3 隐藏问题

上述代码在构造函数中调用虚函数容易被编译器发现，如果写成以下代码，就容易通过编译器的检测，但同样存在一样的问题：

```C%2B%2B
class Transaction {                                // 所有交易的 base class
public:
    Transaction (){
        init();                                    // 调用 non-virtual
    }
    virtual void logTransaction() const = 0;
    ...
private:
    void init(){
        ...
        logTransaction();                           // 这里调用 virtual
    }
};
```

这段代码通常不会引发编译器和连接器异常，由于 logTransaction 是一个 pure virtual 函数，当 pure virtual 函数被调用，大多执行系统会中止程序。如果 logTransaction 是个正常的，它就会被调用。

确定构造函数和析构函数都没有调用 virtual 函数，而且它们调用的所有函数都服从同一约束。

#### 9.4 解决方法

在 class Transaction 内将 logTransaction 函数改为 non-virtual ，然后要求 derived class 构造函数传递必要信息给 Transaction 构造函数，而后那个构造函数便可以安全地调用 non-virtual logTransaction。

```C%2B%2B
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& );     // 现在是个 non-virtual 函数
};

Transaction::Transaction(const std::string& logInfo){
    ...
    logTransaction(logInfo);                     // 如今是个 non-virtual 调用
}

class BuyTransaction: public Transaction {          // derived class
public:
    BuyTransaction(parameters):Transaction(createLogString(parameters))
    { ... }                         // 将 log 信息传给 base class 构造函数
    ...
private:
    static std::string createLogString(parameters);
};
```

createLogString 作为一个静态函数，就不可能意外指向 “初期未成熟之 BuyTransaction 对象内尚未初始化的成员变量。”

换句话话说，不能使用 virtual 函数从 base classes 向下调用，在构造期间，可以“令 derived classes 将必要的构造信息向上传递至 base class 构造函数” 替换而加以弥补。

#### 9.5 总结

在构造和析构期间不要调用 virtual 函数，因为这类调用从不下降至 derived class。

### 条款 10：令 operator= 返回一个 reference to *this

关于赋值，可以将它们写成连锁形式：

```C%2B%2B
int x,y,z;
x = y = z = 15;         // 赋值连锁形式
```

有趣的是，赋值采用右结合律，所以上述赋值被解析为：

```C%2B%2B
x = (y = (z = 15));
```

为了实现“连锁赋值”，赋值操作符必须返回一个 reference 指向操作符的左侧实参。这是为 classes 实现赋值操作符时应该遵循的协议

```C%2B%2B
class Widget {
public:
    ...
    Widget& operator=(const Widget& rhs){        // 返回类型是个 reference，指向当前对象
        ... 
        return *this;                            // 指向左侧对象
    } 
};
```

这个协议不仅适用于以上的标准赋值形式，也适用于所有赋值相关运算，例如：

```C%2B%2B
class Widget{
public:
    ...
    Widget& operator+=(const Widget& rhs){    // 这个协议适用于 += -= *= 等等
        ...
        return *this;
    }
    
    Widget& operator=(int rhs){              // 此函数也适用于，即使此一操作符的参数类型不符合协定
        ...
        return *this;
    }
};
```

这只是协议，并无强制性。这份协议被所有的内置类型和标准程序库提供的类型如 string, vector, complex, std::shared_ptr 共同遵守。

#### 总结

令赋值（assignment） 操作符返回一个 reference to *this。

### 条款 11：在 operator= 中处理 “自我赋值”

“自我赋值”发生在对象被赋值给自己时：

```C%2B%2B
class Widget { ... };
Widget w;
...
w = w;
```

有些自我赋值并不容易被发现：

```C%2B%2B
a[i] = a[j];           // 潜在的自我赋值，如果 i 和 j 有相同的值
*px = *py;             // 潜在的自我赋值，如果 px 和 py 恰巧指向同一个东西
```

实际上两个对象只要来自同一个继承体系，它们甚至不需要声明为相同类型就可能造成 “别名”，因为一个 base class 的 reference 或 pointer 可以指向一个 derived class 对象：

```C%2B%2B
class Base {};
class Derived:public Base { ... };
void doSomething(const Base& rb, Derived* pd);  // rb 和 *pd 有可能其实是同一对象
```



如果尝试写一个资源管理的 class 就得时刻注意 “自我赋值”问题，防止在停止使用资源之前就意外释放了它。

假设现在需要建立一个 class 用来保存一个指针指向一块动态分配的位图(bitmap)

```C%2B%2B
class Bitmap { ... };
class Widget {
    ...
private:
    Bitmap *pb;           // 指针，指向一个从 heap 分配而得到的对象
};
```

下面的 operator= 实现代码，表面上看起来合理，但自我赋值出现时并不安全。

```C%2B%2B
Widget&
Widget::operator=(const Widget& rhs)      // 一份不安全的 operator= 实现版本
{
    delete pb;            // 停止使用当前的 bitmap
    pb = new Bitmap(*rhs.pb); // 使用 rhs's bitmap 副本
    return *this;          
}
```

上述代码存在的问题是：operator= 函数内的 *this 和 rhs 有可能是同一个对象。如果真是这样，delete 就不只是销毁当前对象的 bitmap，它也销毁 rhs 的 bitmap。最后返回的 this 指针就可能指向一个已经删除的对象。

#### 11.1 解决方案

欲阻止这种错误，传统的做法是在 operator= 最前面设置一个 “证同测试”达到 “自我赋值”的检验目的：

```C%2B%2B
Widget& Widget::operator=(const Widget& rhs){
    if(this == &rhs) return *this;      // 证同测试，如果是自我赋值就不做任何事
    
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```

上述代码行得通，但不具备 “异常安全性”。如果 "new Bitmap" 导致异常（无论是分配时内存不足或因为 Bitmap 的 copy 构造函数抛出异常），Widget 最终会持有一个指针指向一块被删除的 Bitmap，这样的指针有害。

令人高兴的是，让 operator= 具备 “异常安全性” 往往自动获得 “自我赋值安全”的回报。因此越来越多人倾向于不去管它，把焦点放在实现 “异常安全性”(exception safety)上。

许多时候一群精心安排的语句就可以导出异常安全（以及自我赋值安全）的代码，这就够了。例如以下代码：我们只需注意在复制 pb 所指东西之前别删除 pb。

```C%2B%2B
Widget& Widget::operator=(const Widget& rhs){
    Bitmap* pOrig = pb;        // 记住原先的 pb
    pb = new Bitmap(*rhs.pb);  // 令 pb 指向一个副本
    delete pOrig;              // 删除原先的 pb
    return *this;
}
```

现在，如果 "new Bitmap" 抛出异常，pb（及其栖身的那个 Widget）保持原状。

你也可以把 “证同测试”再次放回函数起始处，然而这样做之前先问问自己，这样的“自我赋值”的发生频率有多高，因为这项测试也需要成本，它会使代码变大，并导入一个新的控制流分支，两者都会降低执行速度。

#### 11.2 copy and swap

上述解决方案的一个替代方案称为 copy and swap 技术，这个技术和 “异常安全性”有密切关系。这也是一个常见而够好的 operator= 编写办法。

```C%2B%2B
class Widget {
...
void swap(Widget& rhs);             // 交换 *this 和 rhs 的数据
...
};

Widget& Widget::operator=(const Widget& rhs){
    Widget temp(rhs);             // 为 rhs 数据制作一份复件
    swap(temp);                    // 将 *this 数据和上述复件数据交换
    return *this;
}
```

这个主题的另一个变奏曲利用了以下事实：

1. 某 class 的 copy assignment 操作符可能被声明为 “以 by value 方式接受实参”
2. 以 by value 方式传递东西会造成一份复件

```C%2B%2B
Widget& Widget::operator=(Widget rhs)     // rhs 是被传对象的一份复件
{
    swap(rhs);
    return *this;
}
```

上述代码牺牲了清晰性，然而将 "copying 动作" 从函数本体移至 “函数参数构造阶段” 却可令编译器有时生成更高效的代码。

#### 11.3 总结

- 确保当对象自我赋值时 operator= 有良好行为。其中技术包括：比较“来源对象”和“目标对象”的地址，精心周到的语句顺序，以及 copy-and-swap
- 确定任何函数如果操作一个以上的对象，而其中多个对象是同一对象时，其行为仍然正确。

### 条款 12：复制对象时勿忘其每一个成分

如果你为 class 添加一个成员变量，就必须同时修改 copying 函数（包括 拷贝构造函数和 copy assignment 运算符）。你也需要修改 class 的所有构造函数以及任何非标准形式的 operator= 。如果你忘记了，编译器不太可能提醒你。

一旦发生继承，可能会造成更大的危机。因为大部分人的 copying 函数（包括 拷贝构造函数和 copy assignment 运算符）可能会忽略 base class 的成员。

任何时候只要你承担起 “为 derived class 编写 copying 函数”的重大责任，必须很小心地复制其 base class 成分。那些成分往往是 private ，所以你无法直接访问它们，应该让derived class 的 copying 函数调用相应的 base class 函数。

本条款题目所说的“复制每一个成分”应该很清楚了。当编写一个 copying 函数，请确保：

1. 复制所有 local 成员变量。
2. 调用所有 base classes 内的适当 copying 函数。

copy 构造函数和 copy assignment 运算符有着相同的实现本体，这有可能诱使你让某个函数调用另一个函数以避免代码重复。记住：让某个 copying 函数调用另一个 copying 函数无法让你达到想要的目标。

当你发现 copy 构造函数和 copy assignment  操作符有着相似的代码，消除代码重复的做法是，建立一个新的成员函数给两者调用，这样的函数往往是 private 而且常被命名为 init。

#### 总结

Copying 函数应该确保复制“对象内的所有成员变量”及“所有 base class 成分”。

不要尝试以某个 copying 函数实现另一个 copying 函数，应该将它们的共同成分放进第三个函数中，并由两个 copying 函数共同调用。



## 三、资源管理

所谓资源就是，一旦用了它，将来必须还给系统。C++ 程序中最常使用的资源就是动态分配内存。但是内存只是你必须管理的众多资源之一，其他常见的资源还包括文件描述符(file descriptors)，互斥锁(mutex locks)，图形界面中的字型和笔刷，数据库连接，以及网络 sockets。无论哪种资源，当你不再使用它时，就必须归还给系统。

### 条款 13：以对象管理资源

- 获得资源后立刻放进管理对象内。实际上，“以对象管理资源”的观念常被称为“资源获取即初始化”（RAII）。获得一笔资源后于同一语句内以它初始化某个管理对象，有时候获得的资源被拿来赋值某个对象。无论何种做法，每一笔资源都在获得的同时立刻被放进管理对象中。
- 管理对象运用析构函数确保资源被释放。不论控制流如何离开区块，一旦对象被销毁其析构函数自然会被自动调用，于是资源被释放。资源的释放动作可能发生异常，不过条款 8 已经能够解决这个问题。

#### 13.1 auto_ptr

auto_ptr 在 C++ 11 中已经被弃用，被 std::unique_ptr 取代 ，后者具有类似功能的新设施，具有改进的安全性

许多动态资源被动态分配于 heap 内而后被用于单一区块或函数内，它们应该在控制流离开那个区块或函数时被释放。标准库提供的 auto_ptr 正是这种情形下的产物。

auto_ptr 被称为智能指针，其析构函数自动对其所指对象调用 delete。它的使用方法如下：

```C%2B%2B
class Investment { ... };
// 作为工厂函数，返回指针，指向 Investment 继承体系内的动态分配对象，调用者有责任删除它
Investment* createInvestment();

void f(){
    std::auto_ptr<Investment> pInv(createInvestment()); 
    // 调用 factory 函数
    // 一如以往地使用 pInv
    // 经由 auto_ptr 的析构函数自动删除 pInv                                                      
}
```

auto_ptr 被销毁时自动删除它所指之物，别让多个 auto_ptr 同时指向同一对象。为了预防这个问题，auto_ptr 具有不寻常的性质，如果通过 copy 构造函数或 copy assignment 操作符 复制它们，它们就会变成 null，而复制所得的指针将取得资源的唯一拥有权。

```C%2B%2B
std::auto_ptr<Investment> pInv1(createInvestment());    //pInv1 指向 createInvestment 返回物
std::auto_ptr<Investment> pInv2(pInv1); // 现在 pInv2 指向对象，pInv1 被设未 null
ppInv1 = pInv2;   // 现在 pInv1 指向对象，pInv2 被设为 null               
```

#### 13.2 shared_ptr

auto_ptr 的替代方案是 “引用计数型智能指针”（RCSP），一个 shared_ptr 对象会跟踪共有多少对象指向某笔资源，并在无人指向它时自动删除该资源。

RCSP 有点类似于垃圾回收，但它无法打破环状引用，例如两个其实已经没被使用的对象彼此互指，因而好像还处于“被使用”状态。

```C%2B%2B
void f(){
    std::shared_ptr<Investment> pInv(createInvestment()); 
    // 调用 factory 函数
    // 一如以往地使用 pInv
    // 经由 shared_ptr 的析构函数自动删除 pInv                                                      
}
```

以下代码和 auto_ptr 的版本相同，但复制行为正常多了：

```C%2B%2B
void f(){
    std::shared_ptr<Investment> pInv1(createInvestment());    //pInv1 指向 createInvestment 返回物
    std::shared_ptr<Investment> pInv2(pInv1); // 现在 pInv2 和 pInv1 指向同一对象
    ppInv1 = pInv2;   // 同上，无任何改变
    ...
}   // pInv1 和 pInv2 被销毁，它们所指的对象就会iu被自动销毁
```

由于 shared_ptr 正常的复制行为，它们可被用于 STL 容器等。本条款并不针对 auto_ptr ，shared_ptr，只是强调 “以对象管理资源的重要性”

#### 13.3 动态数组

auto_ptr 和 std::shared_ptr 都在其析构函数内做 delete 而不是 delete[] 动作。意味着不能在动态分配而得到的 array 身上使用 auto_ptr 或 std::shared_ptr。可叹的是，它们仍能够通过编译：

```C%2B%2B
std::auto_ptr<std::string> aps(new std::string[10]);  // 搜主意，会用上错误的 delete 形式
std::shared_ptr<int>spi(new int[1024]);        // 相同问题
```

你会惊讶的发现，C++ 并没有针对“动态分配数组”而设计的类似 auto_ptr 或 std::shared_ptr。因为 vector 和 string 几乎可以取代动态分配而得到的数组。

如果你觉得拥有针对数组而设计的，类似 auto_ptr 和 std::shared_ptr 那样 classes 较好。那就看看 Boost 吧，boost::scoped_array 和 boost::shared_array classes 提供了你想要的行为。

使用手工释放资源，例如使用 delete 而不是一个资源管理类，容易发生某些错误。

有时候需要自己实现资源管理类，就需要你精巧地制作你自己的资源管理类。

#### 13.4 总结

- 为防止资源泄漏，请使用 RAII 对象，它们在构造函数中获得资源并在析构函数中释放资源。
- 两个常被使用的 RAII classes 分别是 std::shared_ptr 和 auto_ptr。前者通常是较佳选择，因为其 copy 行为比较直观。若选择 auto_ptr，复制动作会使它指向 null。



### 条款14：在资源管理类中小心 coping 行为

类似 auto_ptr 和 std::shared_ptr 比较适合位于 heap 内存的资源。除此以外的其他资源，可能需要建立自己的资源管理类。

#### 14.1 互斥锁

假设现在使用 C API 函数处理类型为 Mutex 的互斥锁对象，共有 lock 和 unlock 两个函数可用：

```C%2B%2B
void lock(Mutex* pm);             // 锁定临界区
void unlock(Mutex* pm);           // 解锁临界区
```

为了确保不会忘记将一个临界区解锁，可以建立一个 class 来管理锁。这样的 class 的基本结构由 RAII 守则支配，业就是“资源在构造期间获得，在析构期间释放”：

```C%2B%2B
class Lock{
public:
    explicit Lock(Mutex* pm):mutexPtr(pm)
    {lock(mutexPtr);}          // 获得资源
    
    ~Lock() { unlock(mutexPtr); }  // 释放资源
private:
    Mutex *mutexPtr;
};
```

用户对 Lock 的用法符合 RAII 方式：

```C%2B%2B
Mutex m;         // 定义你需要的互斥锁
...
{              // 建立一个区块用来定义 critical section(临界区)
    Lock ml(&m);    // 锁定临界区
    ...             // 执行 critical section 内的操作
        
}                  // 在区块末尾，自动解锁临界区
```

#### 14.2 Coping 行为

上面工作的很好，但如果 Lock 对象被复制，会发生什么？

```C%2B%2B
Lock ml1(&m);                 // 锁定 m
Lock ml2(ml1);                // 将 ml1 复制到 ml2 身上，这会发生什么？
```

这是某个一般化问题的特殊例子。这个一般化问题是每一位 RAII class 作者一定需要面对的，当一个 RAII 对象被复制，大多数情况有以下两种选项：

- 禁止复制：许多时候允许 RAII 对象被复制并不合理。比如像 Lock 这样的 class，因为很少能够合理拥有 “同步化基础器物”的复件。

如果复制动作对 RAII class 并不合理，就应该禁止它，条款 6 说了怎么做，将 copying 操作声明为 private，对 Lock 而言看起来这样：

```C%2B%2B
class Lock: private Uncopyable {           // 禁止复制
public:
    ...
};
```

- 对底层资源采用 “引用计数法”：有时希望保有资源，直到它的最后一个使用者被销毁。这种情况下复制 RAII 对象时，应该将资源的“被引用数”递增。类似于 std::shared_ptr

通常只要内含一个 std::shared_ptr 成员变量，RAII classes 便可以实现 reference-counting copying 行为。将前述 Lock 改为使用 reference counting，它可以改变 mutexPtr 的类型，将它从 Mutex* 改为 std::shared_ptr<Mutex>，

可惜它的缺省行为是“当引用计数为 0 时删除其所指物”，那不是我们所要的行为，对于 Mutex，我们想要的释放动作是解除锁定而非删除。

幸运的是 std::shared_ptr 允许指定所谓的 “删除器”(deleter)，那是一个函数或函数对象。

```C%2B%2B
class Lock{
public:
    explicit Lock(Mutex* pm):mutexPtr(pm,unlock) // 以某个 Mutex 初始化 shared_ptr，并以 unlock 函数为删除器
    {
        lock(mutexPtr.get());    // 条款 15 谈到 “get”
    }
private:
    std::shared_ptr<Mutex> mutexPtr;  // 使用 shared_ptr 替换 raw pointer
};
```

注意，本例中不再声明析构函数，因为没有必要。条款 5 说过，class 析构函数（无论是编译器生成的，或用户定义的）会自动调用其 non-static 成员变量的析构函数，对于上述例子而言，就是调用我们指定的删除器 unlock。

- 复制底部资源

只要你喜欢，可以针对一份资源拥有其任意数量的复件。而你需要“资源管理类”的唯一理由是，当你不再需要某个附件时确保它被释放。在这种情况下复制资源管理对象，应该同时也复制其所管理的资源。

例如，某些标准字符串类型是由“指向 heap 内存”的指针构成（那内存被用来存放字符串的组成字符）。这种字符串对象内含一个指针指向一块 heap 内存。当这样一个字符串对象被复制，不论指针或其所指内存都会被制作出一个复件。这样的字符串展现深度复制行为，即深拷贝。

- 转移底部资源的拥有权

某些罕见场合下可能希望永远只有一个 RAII 对象指向一个未加工资源，即使 RAII 对象被复制依然如此。此时资源的拥有权会从被复制物转移到目标物，这点类似于 auto_ptr 奉行的复制意义。

#### 14.3 总结

- 复制 RAII 对象必须一并复制它所管理的资源，所以资源的 copying 行为决定 RAII 对象的 copying 行为。
- 普遍而常见的 RAII class copying 行为是：抑制 copying，采用引用计数法。不过其他行为也可能被实现。

### 条款 15：在资源管理类中提供对原始资源的访问

资源管理类保证资源的正确释放，避免了资源泄漏。许多 APIs 需要直接使用资源，因此一个资源管理类也应该提供访问原始资源的方法。

```C%2B%2B
std::shared_ptr<Investment>pInv(createInvestment()); 
int daysHeld(const Investment* pi);        // 返回投资天数
```

当这样调用的时候：

```C%2B%2B
int days = daysHeld(pInv);          // 错误
```

这样调用是错误的，因为 daysHeld 需要的是指针，但传给它的是 std::shared_ptr<Investment> 对象 。这时候需要一个函数可将 RAII class 对象转换为其所含之原始资源。有两个办法可以达成目标：显式转换和隐式转换

std::shared_ptr 和 auto_ptr 都提供一个 get 成员函数，用来执行显式类型转换，也就是它会返回智能指针内部的原始指针。

几乎所有的智能指针，包括 std::shared_ptr 和 auto_ptr 也重载了指针取值操作符（operator-> 和 operator*）,它们允许隐式转换至底部原始指针。

```C%2B%2B
class Investment{
public:
    bool isTaxFree() const;
    ...
};
Investment* createInvestment();      // factory 函数
std::shared_ptr<Investment>pi1(createInvestment());
bool taxable1 = !(pi1->isTaxFree());    // 经由 operator-> 访问资源
bool taxable2 = !((*pi1).isTaxFree());  // 经由 operator* 访问资源
```

下面是用于字体的 RAII class

```C%2B%2B
FontHandle getFont();
void releaseFont(FontHandle fh);   // C API   
  
class Font{          // RAII class
public:
    explicit Font(FontHandle fh):f(fh){}    // 获得资源 采用 pass-by-value
    ~Font() { releaseFont(f);}              // 释放资源
private:
    FontHandle f;                           // 原始资源
};
```

#### 15.1 显式类型转换

假设有大量与字体相关的 C API，它们处理 FontHandles，那么“将 Font 对象转换为 FontHandle ”会是很频繁的需求。Font class 可为此提供一个显式转换函数，如下：

```C%2B%2B
class Font{
public:
    ...
    FontHandle get() const { return f;};    // 显式转换函数
    ...
};
```

使用这个方法，需要 FontHandle 的每个地方都需要调用 get 函数。

####  15.2 隐式类型转换

```C%2B%2B
class Font{
public:
    ...
    operator FontHandle() const       // 隐式转换函数
    {return f;}
    ...
};
```

这样当客户需要使用原始资源的时候就比较轻松自然：

```C%2B%2B
Font f(getFont());
int newFontSize;
...
changeFontSize(f,newFontSize);          // 将 Font 隐式转换为 FontHandle
```

上述隐式转换会增加犯错误的机会。例如客户可能会在需要 Font 时意外创建一个 FontHandle：

```C%2B%2B
Font f1(getFont());
...
FontHandle f2 = f1;          // 原义要拷贝一个 Font 对象，却反而将 f1 隐式转换为 FontHandle 然后才复制它
```

选择隐式类型转换还是显式类型转换取决于 RAII class 被设计执行的特定工作。

通常显式转换函数如 get 是比较受欢迎的路子，因为它将“非故意之类型转换”的可能性最小化了。

#### 15.3 总结

- APIs 往往需要访问原始资源，所以每一个 RAII class 应该提供一个“取得其管理之资源”的办法。
- 对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。

### 条款 16：成对使用 new 和 delete 时要采取相同形式

考虑以下代码：

```C%2B%2B
std::string* stringArray = new std::string[100];
...
delete stringArray;
```

看似井然有序，使用了 new，也搭配了对应的 delete。但还是有某样东西完全错误，该程序行为不明确。当使用 delete 回收内存的时候应该指明是回收单一对象，还是一个动态数组。

单一对象和对象数组的结构不完全一样，因为对象数组相比单一对象，其内存布局还包括“数组大小的记录”。

如果 new 和 delete 没有采用相同的形式，会造成以下未定义行为：

- new 对象，delete 对象数组：delete 可能会读取若干内存并将其解释为“数组大小”。
- new 对象数组，delete 对象：结果亦未定义，可能导致更少的析构函数被调用。                                                                                                                                                                                                                                                                                                                                                                          

#### 16.1 typedef

对于喜欢 typedef 的人更应该注意，当程序员以 new 创建该种 typedef 类型对象时，该以哪一种 delete 形式删除之。考虑下面的代码：

```C%2B%2B
typedef std::string AddressLines[4];     
std::string* pal = new AddressLines;     // 该语句就像使用 new string[4] 一样
```

必须匹配“数组形式”的delete：

```C%2B%2B
delete pal;           // 行为未定义
delete []pal;         // 正确
```

为避免类似错误，最好不要对数组形式做 typedef 动作。标准库提供了 string，vector 等 templates，可将数组的需求降至几乎为零。例如本例中 AddressLines 可定义为 “由 strings 组成的一个 vector”，也就是类型 vector<string>

#### 16.2 总结

如果在 new 表达时中使用 []，必须在相应的 delete 表达式也使用[] 。如果没有在 new 表达式不使用 []，一定不要在相应的 delete 表达式中使用 []。



### 条款17：以独立语句将 newed 对象置入智能指针

#### 17.1 资源泄漏问题

假设有以下代码：

```C%2B%2B
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);
```

现在考虑调用 processWidget：

```C%2B%2B
processWidget(new Widget, priority());
```

上面对 processWidget  的调用是错误的，它甚至不能通过编译，std::shared_ptr 构造函数需要一个原始指针，但该构造函数是个 explicit 构造函数，无法进行隐式转换。如果写成以下这样就可以通过编译：

```C%2B%2B
processWidget(std::shared_ptr<Widget>(new Widget),priority());
```

表面上看没有问题，但其实上述调用可能泄漏资源。

编译器产出一个 processWidget 调用码之前，必须先首先核算即将被传递的各个实参：

第一实参 std::shared_ptr<Widget>(new Widget) 由两部分组成

- 执行 "new Widget" 表达式
- 调用 std::shared_ptr 构造函数

于是在调用 processWidget 之前，编译器必须创建代码，做以下三件事：

- 调用 priority
- 执行 "new Widget"
- 调用 std::shared_ptr 构造函数

C++ 编译器以什么次序完成这些事情呢？弹性很大。这和其他语言如 Java 和 C# 不同，那两种语言总是以特定次序完成函数参数的核算。可以确定的是 "new Widget" 一定执行于 std::shared_ptr 构造函数之前，但对 priority 的调用则可以排在第一或第二或第三执行。其中可能的执行次序如下：

1. 执行 "new Widget"
2. 调用 priority
3. 调用 std::shared_ptr 构造函数

现在想想，万一对 priority 的调用导致异常，会发生什么？在这种情况下  "new Widget" 返回的指针将会遗失，因为它未被置入 std::shared_ptr  内。所以在对 processWidget 的调用过程，可能引发资源泄漏。因为在“资源创建”和“资源被转换为资源管理对象”两个时间点之间可能发出异常干扰。

#### 17.2 解决办法

使用分离语句，分别写出 （1）创建 Widge （2）将它置入一个智能指针，然后再把那个智能指针传给 ProcessWidget：

```C%2B%2B
std::shared_ptr<Widget>pw(new Widget);  // 在单独对象内以智能指针存储 newed 所得对象
processWidget(pw, priority());         // 这个调用动作绝不至于造成泄漏
```

编译器对于“跨越语句的各项操作”没有重新排列的自由（只有在语句内它才拥有那个自由度）

#### 17.3 总结

以独立语句将 newed 对象存储于智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。



## 四、设计与声明

所谓软件设计，是“令软件做出你希望它做的事情”的步骤和做法，通常以颇为一般性的构想开始，最终演变成十足的细节，以允许特殊接口的开发。这些接口而后必须转换为 C++ 声明式。

### 条款18：让接口容易被正确使用，不易被误用

#### 18.1 接口误用

C++ 的接口类型主要有以下几种：function 接口，class 接口，template 接口。你设计的接口应该容易被正确使用，如果客户对你所提供的接口的用法错误，你至少也得负一部分责任。

假设有以下接口：

```C%2B%2B
class Date {
public:
    Date(int month, int day, int year);
};
```

看似挺合理，但客户很容易犯以下错误：

1. 他们可能以错误的次序传递参数

```
Date d(30 , 3, 1995);   // 应该是 "3,30" 而不是 "30, 3"  
```

1. 他们可能传递一个无效的月份或天数

```
Date d(2, 31, 2022);    // 应该是 "3,31" 而不是 “2，31”    
```

#### 18.2 解决方法

许多错误可以通过导入新类型而获得预防，对于上述问题我们可以通过导入简单的外覆类型（wrapper types） 来区别天数，月份和年份，然后于 Date 构造函数中使用这些类型：

```C%2B%2B
struct Day{
explicit Day(int d):val(d){}
int val;
};

struct Month{
explicit Month(int m):val(m){}
int val;
};

struct Year{
explicit Year(int y):val(y){}
int val;
};

class Date{
public:
    Date(const Month& m, const Day& d, const Year& y);
    ...
};

Date d(30, 3. 1995);                          // 错误，不正确类型
Date d(Day(30), Month(3), Year(1995));        // 错误，不正确类型
Date d(Month(3), Day(30), Year(1995));        // 正确
```

当正确的类型就位，限制其值有时候也是合理的。例如一年只有 12 个有效月份，比较安全的解法就是预先定义所有有效的 Months：

```C%2B%2B
class Month{
public:
    static Month Jan() { return Month(1); }
    static Month Feb() { return Month(2); }
    ...
    static Month Dec() { return Month(12); }
    ...
private:
    explicit Month(int m);            // 阻止生成新的月份，这是月份专属数据
    ...
};
Date d(Month::Jan(), Day(31), Year(2022));
```

#### 18.3 资源泄漏

观察以下 factor 函数，它返回一个指针指向 Investment 继承体系内的一个动态分配对象：

```
Investment* createInvestment();
```

上述代码给了用户两次犯错误的机会：

1. 没有删除指针
2. 删除同一个指针超过一次

较佳的接口如下：

```
std::shared_ptr<Investment> createInvestment();
```

这边强迫客户将返回值存储于一个 std::shared_ptr 内，几乎消除了忘记删除底部 Investment 对象的可能性。

#### 18.4 总结

- 好的接口很容易被正确使用，不容易被误用。
- “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
- “阻止误用”的办法包括建立新类型，限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
- std::shared_ptr 支持定制删除器，这可防止 DLL 问题，可被用来解除互斥锁。



### 条款 19：设计 class 犹如设计 type

几乎每一个 class 都要求你面对以下提问：

- 新 type 的对象应该如何被创建和销毁？：这会影响构造函数，析构函数，内存分配函数和释放函数的设计。
- 对象的初始化和对象的赋值该有什么样的差别？：这会影响构造函数和 赋值（assignment）操作符的行为。
- 新 type 如果被 passed by value （以值传递），意味着什么？：copy 构造函数用来定义一个 type 的 pass-by-value 该如何实现。
- 什么是新 type 的 “合法值”？
- 你的新 type 需要配合某个继承图系吗？ 如果新 type 继承自某些既有 classes，就会受到那些 classes 的设计的束缚，特别是受到 “它们的函数是 virtual 或 non-virtual ”的影响。如果你允许其他 classes 继承你的 class，那会影响你所声明的函数，由其是析构函数。
- 你的新 type 需要什么样的转换？
- 什么样的操作符和函数对此新 type 而言是合理的？
- 什么样的标准函数应该驳回？   将需驳回函数声明为 private
- 谁该取用新 type 的成员？也就是成员变量的权限控制
- 什么是新 type 的“未声明接口”？它对效率，异常安全性，以及资源运用提供何种保证？你在这方面提供的保证将为你的 class 实现代码加上相应的约束条件。
- 新 type 有多么一般化？ 考虑使用 class template
- 你真的需要一个新的 type 吗？可能定义一个或多个 non-member 函数或 templates 更能达到目标。



#### 总结

Class 的设计就像 type 的设计。在定义一个新 type 之前，请确定已经考虑过本条款覆盖的所有讨论主题。



### 条款20：宁以 pass-by-reference-to-const 替换 pass-by-value

#### 20.1 对象切割问题

当一个 derived class 对象以 by value 方式传递并被视为一个 base class 对象，base class 的 copy 构造函数会被调用，而“造成此对象像个 derived class 对象”的那些特质化性质全部被切割调了，仅仅留下一个 base class 对象。

```C%2B%2B
void printNameAndDisplay(Window w){       // 不正确，参数可能被切割
    std::cout<<w.name();
    w.display();
}
```

解决对象切割问题的办法，就是以 by reference-to-const 的方式传递 w

```C%2B%2B
void printNameAndDisplay(const Window& w){       // 很好，参数不会被切割
    std::cout<<w.name();
    w.display();
}
```

现在传进来什么类型，w 就表现出那种类型。

一般而言，可以合理假设 "pass-by-value 并不昂贵"的唯一对象就是内置类型和 STL 的迭代器和函数对象。至于其他任何东西都请遵守本条款的忠告，尽量以 pass-by-reference-to-const 替换 pass-by-value



#### 20.2 总结

尽量以 pass-by-reference-to-const 替换 pass-by-value。前者通常比较高效，并可以避免切割问题。

以上规则并不适用内置类型，以及 STL的迭代器和函数对象。对它们而言，pass-by-value 往往比较适当。



### 条款 21：必须返回对象时，别妄想返回其 reference

避免传递一个 references 指向其时并不存在的对象。

所谓 reference 只是个名称，代表某个既有对象，任何时候看到一个 reference 声明式，都应该问自己，它的另一个名称是什么。

避免写出以下糟糕代码：

```C%2B%2B
const Rational& operator*(const Rational& rhs){
    static Rational result;         // static 对象，此函数将返回其 reference
    result = ...;                   // 将 rhs 乘以当前对象，并将结果置于 result
    return result;
}
```

以下是完全合理的客户代码：

```C%2B%2B
bool operator==(const Rational& rhs);       // 一个针对 Rational 而写的 operator==
Rational a,b,c,d;

if((a*b)==(c*d)){
    当乘积相等，做适当的动作
}else{
    当乘积不相等，做适当的动作
}
```

表达式  ((a*b)==(c*d)) 总是被核算为 true，无论 a,b,c 和 d，因为 a*b 和 c*d 的操作结果都指向同一个 static 对象。

#### 总结

绝不要返回 Pointer 或 reference 指向一个 local stack 对象，或返回 reference 指向一个 heap-allocated 对象，或返回 Pointer 或 reference 指向一个 local static 对象而有可能同时需要多个这样的对象



### 条款 22：将成员变量声明为 private

成员变量应该是 private。

将成员变量声明为 private 意味着，访问某个成员变量需要通过成员函数实现，以下是这么做的好处：

#### 22.1 权限控制

使用函数可以对成员变量的处理有更精确的控制，包括只读，只写，读写等。

#### 22.2 封装

将成员变量隐藏在函数接口的背后，可以为“所有可能的实现”提供弹性。如果通过函数访问成员变量，日后可以改变某个计算替换这个成员变量，而 class 客户一点也不会知道 class 的内部已经起了变化。

public 意味着不封装，不封装意味着不可改变。即使拥有 class 的原始码，改变任何 public 事物的能力还是极端受到束缚，因为那会破坏太多客户端。

protected 成员变量也类似，假设我们有一个 public 成员变量，而我们最终取消了它，会导致所有使用它的 derived classes 被破坏。

protected 成员变量就像 public 成员变量一样缺乏封装性，因为在这两种情况下，如果成员变量被改变，都会有不可预知的大量代码受到破坏。

记住：从封装的角度看，其实只有两种访问权限：private（提供封装）和其他（不提供封装）。

#### 22.3 总结

切记将成员变量声明为 private。这可赋予客户端访问数据的一致性，可细微划分访问控制，并提供 class 作者以充分的实现弹性。

protected 并不比 public 更具封装性。



### 条款 23：宁以 non-member non-friend 替换 member 函数

#### 23.1 影响封装性的因素

说的比较抽象：

如果某些东西被封装，它就不再可见。愈多东西被封装（成员变量），愈少人可以看到它。而愈少人看到它，我们就有愈大的弹性去改变它，因为我们的改变仅仅直接影响看到改变的那些人。

愈少代码可以看到数据（也就是访问它），愈多的数据可被封装，而我们也就愈能自由地改变对象数据，例如改变成员变量的数量，类型等等。那么如何测量“有多少代码可以看到某一块数据”呢？计算能够访问该数据的函数数量，作为一种粗糙的量测，愈多函数可以访问它，数据的封装性愈低。

因此将 member 函数替换为 non-member non-friend 函数，可以提高对象的封装性。

#### 23.2 提高代码可扩展性

在 C++，比较自然的做法就是让那些从类里抽离出来的 non-member non-friend 函数位于与该类所在的同一个 namespace 内：

```C%2B%2B
namespace WebBrowserStuff{
    class WebBrowser { ... };
    void clearBrowser(WebBrowser& wb);
}
```

这样只是看起来比较自然，namespace 和 class 不同，前者可以跨越多个源码文件而后者不能。一个像WebBrowser 这样的 class 可能拥有大量的便利函数，将它们分门别类可以使代码结构更清晰。分离这些便利函数的最直接做法就是将相关的一组函数分别在一个头文件中声明：

```C%2B%2B
// 头文件 "webbrowser.h" 这个头文件针对 class WebBrowser 自身 以及 WebBrowser 核心机能
namespace WebBrowserStuff {
class WebBrowser { ... }
...               // 核心机能，例如几乎所有客户都需要的 non-member 函数
}

// 头文件 "webbrowserbookmark.h" 
namespace WebBrowserStuff {
    ...     // 与书签相关的便利函数
} 

// 头文件 "webbrowsercookies.h"
namespace WebBrowserStuff {
    ...     // 与cookie相关的便利函数
} 
```

这正是 C++ 标准程序库的组织方式。标准程序库并不是拥有单一，整体，庞大的<C++StandardLibrary> 头文件并在其中内含 std 命名空间内的每一样东西，而是有数十个头文件（<vector> <algorithm <memory>）

将所有便利函数放在多个头文件内但隶属于同一个命名空间，意味着客户可以轻松扩展这一组便利函数，他们需要做的就是添加更多 non-member non-friend 函数到此命名空间。记住，class 定义式对客户而言是不能扩展的。

#### 23.3 总结

宁可拿 non-member non-friend 函数替换 member 函数。这样做可以增加封装性，包裹弹性（packaging flexibility）和机能扩充性。



### 条款 24：若所有参数皆需要类型转换，请为此采用 non-member 函数

假设这样设计有理数 class:

```C%2B%2B
class Rational {
public:
    Rational(int numerator=0,int denominator = 1);    // 可以不为 explicit ，允许 int 隐式转换为 Rational
    int numerator() const;
    int denominator() const;
private:
    ...
};
```

现在要让 Rational 支持诸如加法，乘法等等。很可能你会想在 Rational class 内为有理数实现 operator*：

```C%2B%2B
class Rational {
public:
    ...
    const Rational operator*(const Rational& rhs) const;
};
```

这样就可以这样将两个有理数以最轻松自在的方式相乘：

```C%2B%2B
Rational oneEight(1,8);
Rational oneHalf(1,2);
Rational result = oneHalf * oneEight;   // 很好
result = result * oneEight; 
```

但你还不满足，希望它支持混合运算，如下所式：

```C%2B%2B
result = oneHalf * 2;        // 很好
result = 2*oneHalf;          // 错误
```

为什么错误呢？当以对应的函数形式重写上述两个式子，问题所在就一目了然了：

```C%2B%2B
result = oneHalf.operator*(2);    // 很好
result = 2.operator*(oneHalf);    // 错误
```

很明显，整数 2 并没有相应的 class，也就没有 operator* 成员函数。编译器也会尝试寻找可被以下这般调用的 non-member operator*（也就是在命名空间内或在 global 作用域内）：

result = operator*(2,oneHalf)；  // 错误

#### 24.1 重载 * 运算符

为了解决上述问题，让 operator* 成为一个 non-member 函 ,m数:

```C%2B%2B
class Rational {
    ...                // 不包括 operator*
};

const Rational operator*(const Rational& lhs, const Rational& rhs){
    return Rational(lhs.numerator()*rhs.numerator(),lhs.denominator()*rhs.denominator())
;}

Rational oneFourth(1,4);
Rational result;
result = oneFourth * 2;    // 没问题
result = 2 * oneFourth;    // 万岁，通过编译了
```

这是一个快乐的结局，但还有一个问题？ operator* 是否应该成为 Rational class 的一个 friend 函数呢？

就本例而言是否定的，因为 operator* 可完全由 Rational 的 public 接口完成任务。无论何时如果可以避免 friend 函数就该避免。

#### 24.2 总结 

如果需要为某个函数的所有参数（包括被 this 指针所指向的那个隐喻参数）进行类型转换，那么这个函数必须是个 non-member



### 条款 25：考虑写出一个不抛出异常的 swap 函数

swap 是个有趣的函数，原本它只是 STL 的一部分，后来成为异常安全性编程的脊柱，以及用来处理自我赋值可能性的一个常见机制。

#### 25.1 缺省实现

缺省情况下 swap 动作可由标准程序库提供的 swap 算法完成，其典型实现如下：

```C%2B%2B
namespace std{
    template<typename T>          // std::swap 的典型实现
    void swap(T&a, T&b){
        T temp(a);
        a = b;
        b = tem;
    }
}
```

只要类型 T 支持 copying（copy 构造函数和 copy assignment 操作符）

缺省版本实现十分平淡，它涉及三个对象的复制，对某些类型而言，这些复制动作无一必要。比如 “以指针指向一个对象，内含真正数据”那种类型，这种设计的常见表现形式是所谓的“pimpl手法”(pointer to implementation)，以这种手法设计 Widget class：

```C%2B%2B
class WidgetImpl{
public:
    ...
private:
    int a,b,c;                          // 可能有许多数据
    std::vector<double>v;               // 意味着复制时间很长
    ...
};

class Widget {                // 这个 class 使用 pimpl 手法
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs){    // 复制 widget 时，令它复制其 WidgetImpl 对象
        ...                                  // 关于 operator= 的一般性实现
        *pImpl = *(rhs.pImpl);
        ...
    }
private:
    WidgetImpl* pImpl;           // 所指对象内含 Widget 数据
};
```

一旦要置换两个 Widget 对象值，唯一需要做的就是置换其 pImpl 指针，但缺省的 swap 算法不知道这一点。它不只复制三个 Widgets，还复制三个 WidgetImpl 对象

#### 25.2 全特化版本

我们希望告诉 std::swap，当 widgets 被置换时真正该做的是置换其内部的 pImpl 指针。确切实践这个思路的一个做法是：将 std::swap 针对 Widget 特化。下面是基本构想，还无法通过编译：

```C%2B%2B
namespace std{
    template<>
    void swap<Widget>(Widget& a, Widget& b)  // 这是 std::swap 针对 “T是Widget”的特化版本
    {
        swap(a.pImpl,b.pImpl);
    }
}
```

"template<>" 表示它是 std::swap 的一个全特化（total template specialization）版本，函数名称之后的 “<Widget>”表示这一特化版本系统针对“T 是 Widget ”而设计。换句话说：当一般性的 swap template 施行于 Widgets 身上便会启用这个版本。

通常我们不能改变 std 命名空间内的任何东西，但可以为标准 templates（如 swap）制造特化版本，使他专属于我们自己的 classes（如 Widget）。

 上述代码无法通过编译，原因是它企图访问 a 和 b 内的 pImpl 指针，而它们却是 private，我们可以将这个特化版本声明为 friend，但我们不那么做，而是令 Widget 声明一个名为 swap 的 public 成员函数做真正的置换工作，然后将 std::swap 特化，令他调用该成员函数：

```C%2B%2B
class Widget{
public:
    ...
    void swap(Widget& other){
        using std::swap;
        swap(pImpl,other.pImpl);            // 若要置换 Widgets 就置换其 pImpl 指针
        ...
    }
};

namespace std{
    template<>
    void swap<Widget>(Widget&a, Widget&b){
        a.swap(b);            // 若要置换 Widgets，调用其 swap 成员函数
    }
}
```

以上代码不只能通过编译，还与 STL 容器有一致性，因为所有 STL 容器都提供 public swap 成员函数 和 std::swap 特化版本。

模板特化以后，实际上其本身已经不是templatized，而偏特化，仍然带有templatized。

关于模板的特化与偏特化：https://www.jianshu.com/p/4be97bf7a3b9

#### 25.3 偏特化版本

假设 Widget 和 WidgetImpl 都是 class templates 而非 classes:

```C%2B%2B
template<typename T>
class WidgetImpl { ... };

template<typename T>
class Widget { ... };

namespace std{
    template<typename T>                               // 偏特化 swap
    void swap<Widget<T>>(Widget<T>&a, Widget<T>&b){    // 错误 不合法
        a.swap(b);
    }
}
```

声明一个 non-member swap 让他调用 member swap，但不再将那个 non-member swap 声明为 std::swap 的特化版本或重载版本。不能在 std 命名空间内偏特化 swap，假设 Widget 的所有相关机能都被置于命名空间 WidgetStuff 内：

```C%2B%2B
namespace WidgetStuff{
    ...                  // 模板化的 WidgetImpl 等等
    template<typename T>    // 同前，内含 swap 成员函数
    class Widget { ... };
    ...
    template<typename T>     // non-member swap 函数
    void swap(Widget<T>& a, Widget<T>& b){    // 这里并不属于 std 命名空间
        a.swap(b);
    }
}
```

现在任何地点的任何代码如果打算置换两个 Widget 对象，因而调用 swap，C++ 的名称查找法则(name lookup rules) 会找到 WidgetStuff 内的 Widget 专属版本。如果你想让你的 “class 专属版” swap 在尽可能多的语境下被调用，你需要同时在该 class 所在命名空间内写一个 non-member 版本以及一个 std::swap 特化版本。

#### 25.4 using std::swap

假设现在正在编写一个 function template，其内需要置换两个对象值：

```C%2B%2B
template<typename T>
void doSomething(T& obj1, T& obj2){
    ...
    swap(obj1,obj2);
    ...
}
```

应该调用哪个 swap？是 std 既有的那个一般化版本？还是某个可能存在的特化版本？抑或是一个可能存在的 T 专属版本而且可能栖身于某个命名空间（但当然不可以是 std）。

你希望的是调用 T 专属版本，并在该版本不存在的情况下调用 std 内的一般化版本，下面是你希望发生的：

```C%2B%2B
template<typename T>
void doSomething(T& obj1, T& obj2){
    using std::swap;      // 令 std::swap 在此函数内可用
    ...
    swap(obj1,obj2);      // 为 T 型对象调用最佳 swap 版本
    ...
}
```

一旦编译器看到对 swap 的调用，它便会查找适当的 swap 并调用。C++ 的名称查找法则确保找到 global 作用域或 T 所在的命名空间内的任何 T 专属的 swap。

注意不要以这种方式调用 swap:

std::swap(obj1, obj2);           // 这是错误的 swap 调用方式

这将迫使编译器只认 std 内的 swap，而不太可能调用一个定义于它处的较适当 T 专属版本。



如果 swap 缺省实现版本的效率不足（那几乎总是意味着你的 class 或 template 使用了某种 pimpl 手法），试着做以下事情：

1. 提供一个 public swap 成员函数，让他高效地置换你的类型的两个对象值。
2. 在你的 class 或 template 所在命名空间提供一个 non-member swap，并令它调用上述 swap 成员函数
3. 如果你正在编写一个 class（而非 class template），为你的 class 特化 std::swap，并令它调用你的 swap 成员函数

最后，如果你调用 swap ，请确定包含一个 using 声明式，以便让 std::swap 在你的函数内曝光可见，然后不加任何 namespace 修饰符，赤裸裸地调用 swap。

千万记住，成员版 swap 绝不可抛出异常。那是因为 swap 的一个最好的应用是帮助 classes（和 class templates）提供强烈的异常安全性（exception-safety）保障。

上述约束只施行于成员版！不可施行于非成员版，因为 swap 缺省版本是以 copy 构造函数和 copy assignment 操作符为基础，而一般情况下两者都允许抛出异常。

当你写下一个自定版本的 swap，往往提供的不只是高效率置换对象的办法，而且不抛出异常。一般而言这两个特性是连在一起的，因为高效率的 swaps 几乎总是对内置类型操作（例如 pimpl 手法的底层指针），而内置类型上的操作绝不会抛出异常。

#### 25.5 总结

- 当 std::swap 对你的效率不高时，提供一个 swap 成员函数，并确定这个成员函数不抛出异常
- 如果你提供一个 member swap，也该提供一个 non-member swap 用来调用前者。对于 classes（而非 templates）,也请特化 std::swap
- 调用 swap 时应针对 std::swap 使用 using 声明式，然后调用 swap 并且不带任何“命名空间修饰符”
- 为“用户定义类型”进行 std templates 全特化是好的，但千万不要尝试在 std 内加入某些对 std 而言全新的东西。



## 五、实现

### 条款 26：尽可能延后变量定义式的出现时间

以下函数过早定义变量 "encrypted"

```C%2B%2B
std::string encryptPassword(const std::string& password)
{
    using namespace std;
    string encrypted;
    if(password.length() < MinimumPasswordLength){
        throw logic_error("Password is too short");
    }
    ...                    // 必要动作，例如将一个加密后的密码置入变量 encrypted 内
    return encrypted;
}
```

对象 encrypted 在此函数中并位被完全使用，如果有个异常被丢出，它就没有被真正使用，而它的构造和析构成本就浪费了。



稍微改进的版本，将 encrypted 放到较后位置：

```C%2B%2B
void encrypt(std::string& s);        // 在其中的适当地点对 s 加密

std::string encryptPassword(const std::string& password)
{
    using namespace std;
    if(password.length() < MinimumPasswordLength){
        throw logic_error("Password is too short");
    }
    string encrypted;
    encrypted = password;
    encrypt(encrypted);
    return encrypted;
}
```

以上版本，对于变量 "encrypted" ，它首先调用 default 构造函数，然后调用 copy assignment 运算符。

#### 26.1 最优化版本

以下版本可以避免无意义的 default 构造行为。

```C%2B%2B
std::string encryptPassword(const std::string& password)
{
    using namespace std;
    if(password.length() < MinimumPasswordLength){
        throw logic_error("Password is too short");
    }
    string encrypted(password);
    encrypt(encrypted);
    return encrypted;
}
```

####  26.2 循环内构造对象

下面两个一般性结构，哪一个比较好？

```C%2B%2B
// 方法A，定义于循环外 
Widget w;
for(int i = 0; i < n; ++i){
    w = i 的某个值
    ...
}

// 方法B，定义于循环内
for(int i = 0; i < n; ++i){
    Widget w(i 的某个值);
}
```

- 做法 A：1 个构造函数 + 1 个析构函数 + n 个赋值操作
- 做法 B：n 个构造函数 + n 个析构函数

如果 classes 的一个赋值成本低于一组构造+析构成本，做法 A 更高效，由其当 n 值很大的时候。否则做法 B 或许较好。

做法 A 造成名称 w 的作用域比做法 B 更大，有时对程序的可理解性和易维护性性造成冲突。因此除非明确（1）一个赋值成本低于一组构造+析构成本，（2）你正在处理代码中效率高度敏感的部分。否则，你应该使用做法 B。



### 条款 27：尽量少做转型动作

#### 27.1 C 风格类型转换

被称为旧式转型：

(T)expression                  // 将 expression 转型为 T

函数风格的类型转换：

T(expression)                  // 将 expression 转型为 T

两种形式并无差别，只是小括号的摆放位置不同罢了。

#### 27.2 C++ 风格类型转换

被称为新式转型：

const_cast<T>(expression)

dynamic_cast<T>(expression)

reinterpret_cast<T>(expression)

static_cast<T> (expression)

各有不同目的：

- const_cast 通常用来将对象的常量性移除，它也是唯一有此能力的 C++-style 转型操作符。

```C%2B%2B
const int constant = 1024;
int *p =  const_cast<int*>(&constant);
int& r = const_cast<int&>(constant);
(*p)++;
r++;
std::cout << *p << std::endl;
std::cout << r << std::endl;
```

- dynamic_cast 主要用来执行 “安全向下转型”，用来决定某对象是否归属某继承体系中的某个类型。它是唯一无法由旧式语法执行的动作，也是唯一可能耗费重大运行成本的转型动作。

```C%2B%2B
#include <iostream>
using namespace std;


struct A {
  virtual void f() { cout << "Class A" << endl; }
};


struct B : A {
  virtual void f() { cout << "Class B" << endl; }
};


struct C : A {
  virtual void f() { cout << "Class C" << endl; }
};


void f(A* arg) {
  B* bp = dynamic_cast<B*>(arg);
  C* cp = dynamic_cast<C*>(arg);


  if (bp)
    bp->f();
  else if (cp)
    cp->f();
  else
    arg->f();
};


int main() {
  A aobj;
  C cobj;
  A* ap = &cobj;
  A* ap2 = &aobj;
  f(ap);
  f(ap2);
}
```

输出：

```
Class C
Class A
```

- static_cast 用来强迫隐式转换，功能是把一个表达式转换为某种类型，但没有运行时类型检查来保证转换的安全性。例如将 non-const 对象转为 const 对象，或将 int 转为 double 等等。它也可以用来执行上述多种转换的反向转换，例如将 void* 指针 转换为 typed 指针，将 pointer-to-base 转为 ponter-to-derived

```C%2B%2B
int  i;
float f = 166.71;
i =  static_cast<int>(f);
```



- reinterpret_cast 执行低级转型，将数据以二进制存在形式的重新解释，实际动作(及结果)可能取决于编译器，造成了它不可移植，例如将一个 pinter to int 转型为一个 int。这一类转型在低级代码以外很少见。

```C%2B%2B
#include <iostream>


int main(){
        int address = 0xFF1234;
        int *p = reinterpret_cast<int*>(address);


        void *v = reinterpret_cast<void*>(p);
        
        printf("%0x %p\n",address,p);
        printf("%p\n",v);
         return 0;
}
```



static_cast 为什么比 reinterpret_cast 更安全：

```C%2B%2B
#include <iostream>


class A {
    public:
    int m_a;
};
 
class B {
    public:
    int m_b;
};
 
class C : public A, public B {};


int main(){
        C c;
        printf("%p, %p, %p", &c, reinterpret_cast<B*>(&c), static_cast <B*>(&c));
        return 0;
}
```

输出：

```
0x7ffe1d06b3a0, 0x7ffe1d06b3a0, 0x7ffe1d06b3a4
```

c 的类结构如下：

```Plain%20Text
$1 = {
  <A> = {
    m_a = -8368
  }, 
  <B> = {
    m_b = 32767
  }, <No data fields>}
```

前两个的输出值是相同的，最后一个则会在原基础上偏移4个字节，这是因为static_cast计算了父子类指针转换的偏移量，并将之转换到正确的地址（c里面有m_a,m_b，转换为B*指针后指到m_b处），而reinterpret_cast却不会做这一层转换。



旧式转型仍然合法，但新式转型更受欢迎，原因如下：

1. 它们容易在代码中辨识出来（不论是通过人工辨识或使用工具如 grep），因而得以简化“找出类型系统在哪个地点被破坏”的过程。
2. 转型动作的目标愈窄化，编译器愈可能诊断出错误的运用。



#### 27.3 为什么使用旧式转换

唯一使用旧式转型的时机，当需要调用一个 explicit 构造函数将一个对象传递给一个函数时。

例如：

```C%2B%2B
class Widget {
public:
    explicit Widget(int size);
    ...
};
void doSomeWork(const Widget& w);

doSomeWork(Widget(15));   // 以一个 int 加上 “函数风格” 的转型动作创建一个 Widget
doSomeWork((Widget)15);   // C 风格的强制类型转换
doSomeWork(static_cast<Widget>(15));// 以一个 int 加上 “C++ 风格” 的
                                    // 转型动作创建一个 Widget
```



#### 27.4 类型转换误用

许多程序员错误地相信，转型其实什么都没做，只是告诉编译器把某种类型视为另一种类型。其实，任何一个类型转换（不论是通过转型操作而进行的显式转换，或通过编译器完成的隐式转换）往往真的令编译器编译出运行期间执行的代码。

例如：

```C%2B%2B
int x,y;
...
double d = static_cast<double>(x)/y;      // x 除以 y，使用 浮点数除法
```

将 int 转型为 double 肯定会产生一些代码，因为大部分计算器体系结构中，int 的底层表述不同于 double 的底层表述。

##### 避免基于 C++ 对象的内存布局作任何转型动作

```C%2B%2B
class Base { ... };
class Derived: public Base { ... };
Derived d;
Derived *pd = &d;
Base *pb = pd;             // 隐喻地将 Derived* 转换为 Base*
```

在这种情况下会有个偏移量(offset) 在运行期被施行于 Derived* 指针身上，用以取得正确的 Base* 指针值。

上述例子表明，单一对象（例如一个类型为 Derived 的对象）可能拥有一个以上的地址（例如 “以 Base* 指向它”时的地址和 “以 Derived* 指向它” 时的地址），这在 C 不可能发生，Java 和 C# 也不可能发生这种事，但 C++ 可以。实际上，一旦使用多重继承，这事几乎一直发生者。

所以意味着你通常应该避免做出“对象在 C++ 中如何布局的假设”，当然更不该以此假设为基础执行任何转型动作。例如，将对象地址转型为 char* 指针然后在它们身上进行指针算术，几乎总是导致无定义行为。

例如通过一定的偏移，将 将 Derived* 转换为 Base*。对象的布局方式和它们的地址计算方式随编译器的不同而不同，那意味着“由于知道对象如何布局”而设计的转型，在某一个平台行得通，而在其他平台并不一定行得通。

##### 避免修改派生类中其基类部分

例如许多应用框架都要求 derived classes 内的 virtual 函数代码的第一个动作就是先调用 base class 的对应函数。假设我们由个 window base class 和一个 SpecialWindow derived class，两者都定义了 virtual 函数 onResize。进一步假设 SpecialWindow 的 onResize 函数被要求首先调用 window 的 onResize。以下是实现方式之一，看似是对的，其时是错的。

```C%2B%2B
class Window {
public:
    virtual void onResize(){ ... }     
    ...
};

class SpecialWindow: public Window{
public:
    virtual void onResize(){                 // 将 derived onResize 实现代码
        static_cast<Window>(*this).onResize(); // 将 *this 转型为 window，然后调用其 onResize
                                               // 这不可行  
        ...  // 在这里进行 SpecialWindow 专属行为     
    }
    ...
};
```

上述代码的最初预期应该是，将 *this 转型为 window，对函数 onResize的调用也因此调用了 Window::onResize。但事实上，它调用的并不是当前对象上的函数，而是稍早转型动作所建立的一个 “*this 对象的 base class 成分”的暂时副本身上调用 onResize。

再说一次，上述代码并非在当前对象上调用 Window::onResize 之后又在该对象身上执行 SpecialWindow 专属动作。不，它是在“当前对象之 base class 成分”的副本上调用 Window::onResize，然后在当前对象身上执行 SpecialWindow 专属动作。如果 Window::onResize 修改了对象内容，当前对象其实没被改动，改动的是副本。这会使得当前对象进入一种 “伤残”状态：其 base class 成分的更改没有落实，而 derived class 成分的更改倒是落实了。



解决方法是拿掉转型动作，而不是哄骗编译器将 *this 视为一个 base class 对象。如果只是想调用 base class 版本的 onResize 函数，请这么写：

```C%2B%2B
class SpecialWindow: public Window{
public:
    virtual void onResize(){                 
        Window::onResize();   // 调用 Window::onResize 作用于 *this 身上   
                                              
        ...  // 在这里进行 SpecialWindow 专属行为     
    }
    ...
};
```

这个例子说明，如果你打算转型，这就是一个活脱脱的警告信号：你可能正将局面发展至错误的方向。



#### 27.5 dynamic_cast 

dynamic_cast  的许多实现版本执行速度相当慢。一个很普遍的实现版本是基于 "class 名称的字符串比较"。对四层深的单继承体系内的某个对象执行 dynamic_cast 可能会耗用多达四次的 strcmp 调用，用以比较 class 名称。深度继承或多重继承的成本更高。这些实现版本这么做是为了支持动态连接。

在注重效率的代码中，对 dynamic_cast 保持机敏与猜疑。



当你想在一个你认定为 derived class 对象身上执行 derived class 操作函数，但你的手上却只有一个 “指向 base” 的 pointer 或 reference。有两个一般性做法可以避免这个问题。

1. ###### 存储 derived class 对象指针

使用容器并在其中存储直接指向 derived class 对象的指针（通常是智能指针），如此便消除了“通过 base class 接口处理对象”的需要。假设先前的 Window/SpecialWindow 继承体系中只有 SpecialWindow 才支持闪烁效果，可以这么做：

```C%2B%2B
typedef std::vector<std::shared_ptr<SpecialWindow>> VPSW;
VPSW winPtrs;
...
for(VPSW::iterator iter = winPters.begin(); iter!=winPters.end(); ++iter)
    (*iter)->blink();
```

如果容器中存储的是 Window 类型指针，那么在执行 blink() 函数的时候就需要调用 dynamic_cast 将基类指针转换为派生类指针。如下所示：

```C%2B%2B
for(VPSW::iterator iter = winPters.begin(); iter!=winPters.end(); ++iter){
    if(SpecialWindow *psw = dynamic_cast<SpecialWindow*>(iter->get())){
        psw->bink();
    }   
}
```

当然啦，这种做法无法在同一个容器内存储指针“指向所有可能的Window 派生类”。如果真要处理多种窗口类型，你可能需要多种容器，它们都必须具备类型安全性。



1. ###### 通过 base 接口处理 “所有可能的 Window 派生类”

在 base class 内提供 virtual 函数做你想对各个 window 派生类做的事。举个例子，虽然只有 SpecialWindow 可以闪烁，但可以将闪烁函数声明于 base class 内并提供一份 “什么也没做”的缺省实现，如下所示：

```C%2B%2B
class Window {
public:
    virtual void blink() {}
    ...
};

class SpecialWindow: public Window{
public:
    virtual void blink() { ... };
}

typedef std::vector<std::shared_ptr<SpecialWindow>> VPSW;
VPSW winPtrs;
...
for(VPSW::iterator iter = winPters.begin(); iter!=winPters.end(); ++iter)
    (*iter)->blink();
```

无论哪一种写法——“使用类型安全容器”或“将 virtual 函数往继承体系上方移动”，都并不是放之四海皆准，但在许多情况下它们都提供一个可行的 dynamic_cast 替代方案。



优良的 C++ 代码很少使用转型，我们应该尽可能隔离转型动作，通常是把它隐藏在某个函数内。



#### 27.5 总结

- 如果可以，尽量避免转型。特别是在注重效率的代码中避免 dynamic_cast。如果有个设计需要转型动作，试着发展无需转型的替代设计。
- 如果转型是必要的，试着将它隐藏在某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码。
- 宁可使用 C++-style 转型，不要使用旧式转型。前者很容易辨识出来，而且有个各自的职责。



### 条款 28：避免返回 handles 指向对象内部成分

假设所编写的程序涉及矩形，每个矩形可以由两个点来表示（左上角和右下角），为了让 Rectangle 对象尽可能小，可以不把定义矩形的点放在 Rectangle 对象内，而是放在一个辅助的 struct 内再让 Rectangle 去指向它：

```C%2B%2B
class Point {                      // 这个类用来表示点
public:
    Point(int x, int y);
    ...
    void setX(int newVal);
    void setY(int newVal);
    ...
};

struct RectData {                 // 表示一个矩形
    Point ulhc;                   // ulhc="upper left-hand corner" 左上角
    Point lrhc;                   // lrhc="lower right-hand corner" 右下角
};

class Rectangle {
    ...
private:
    std::shared_ptr<RectData> pData;
};
```

Rectangle 客户端必须能够计算 Rectangle 的范围，所以这个 class 提供 upperLeft 函数和 lowerRight 函数。Point 是用户自定义类型，根据条款 20，我们令这些函数返回 reference，代表底层的 Point 对象：

```C%2B%2B
class Rectangle {
public:
    ...
    Point& upperLeft() const { return pData->ulhc; }
    Point& lowerRight() const {return pData->lrhc; }
};
```

上述代码可通过编译，但却是错误的，实际上它是自我矛盾的。

一方面 upperLeft 和 lowerRight 被声明为 const 成员函数，因为他们的目的是提供客户一个读取 Rectangle 相关坐标的方法，而不是让客户修改 Rectangle。另一方面这两个函数却都返回 reference 指向 private 内部数据，调用者可通过这些 reference 更改内部数据，例如：

```C%2B%2B
Point coord1(0, 0);
Point coord2(100, 100);
const Rectangle rec(coord1, coord2);//rec 是个 const 矩形 从 (0,0) 到 (100,100)
rec.upperLeft().setX(50);  // 变成了 (50,0) 到 (100, 100)
```

upperLeft 的调用者能够使用被返回的 reference（指向 rec 内部的 Point 成员变量）。但 rec 其实应该是不可变的(const)。这个告诉我们：

1. 成员变量的封装性最多只等于 “返回其 reference”的函数的访问级别。本例中虽然 ulhc 和 lrhc 都被声明为 private，但它们实际上却是 public，因为 public 函数 upperLeft 和 lowerRight 传出了他们的 reference
2. 如果 const 成员函数传出一个 reference，后者所指数据与对象自身有关联，而它又被存储于对象之外，那么这个函数的调用者可以修改那笔数据，这正是 bitwise constness 的一个附带结果。

上前的情况都是由于“成员函数返回 reference”,如果它们返回的是指针或迭代器，相同的情况还会发生。reference，指针和迭代器统统都是所谓的 handles（号码牌，用于取得某个对象）

对象的“内部”就是指它的成员变量，但其实不被公开使用的成员函数（被称为 protected 或 private）也是对象“内部”的一部分。因此也应该留心不要返回它们的 handles。

我们上述遇到的问题可以轻松去除，只要在它们的返回类型上加上 const 即可：

```C%2B%2B
class Rectangle {
public:
    ...
    const Point& upperLeft() const { return pData->ulhc; }
    const Point& lowerRight() const {return pData->lrhc; }
};
```

#### 28.1 空悬问题

虽然这些函数的写权力被禁止了，但 upperLeft 和 lowerRight 还是返回了“代表对象内部的” handles，有可能在其他场合带来问题，更明确地说是 dangling handles（空悬的号码牌），这种 handles 所指东西不复存在。这种“不复存在的对象”最常见的来源就是函数返回值。

例如某个函数返回 GUI 对象的外框，这个外框采用矩形表示：

```C%2B%2B
class GUIObject { ... };
const Rectangle boundingBox(const GUIObject& obj); // 以by value 方式返回一个矩形
```

现在客户端可以这么使用这个函数：

```C%2B%2B
GUIObject* pgo;          // 让 pgo 指向某个 GUIObject
...
const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft());//取得一个指针指向外框左上点
```

对 boundingBox 的调用获得一个匿名对象，对该函数的调用语句结束后，该匿名对象将被销毁，其内的 Point 将析构。最终导致 pUpperLeft 指向一个不再存在的对象，也就是说一旦产生 pUpperLeft 的语句结束，pUpperLeft 也就变成空悬指针。

这就是为什么函数如果“返回一个 handle 代表对象内部成分”总是危险的原因，不论这个 handle 是否为 const，也不论那个返回 handle 的成员函数是否为 const。这里的唯一关键是：有个 handle 被传出去，一旦如此就暴露了“handle 比其所指对象更长寿”的风险下。

#### 28.2 例外

并不意味着不可以让成员函数返回 handle，例如 operator[] 就允许访问 string 或 vector 的个别元素。尽管如此，这样的函数毕竟是例外，不是常态。



#### 28.3 总结

避免返回 handle（包括 reference，指针，迭代器）指向对象内部。遵守这个条款可增加封装性，帮助 const 成员函数的行为像个 const，并将发生 "虚吊号码牌"的可能性降至最低。



### 条款 29：为 “异常安全”而努力是值得的

假设有个 class 用来表示带背景图案的 GUI 菜单，这个 class 希望用于多线程环境，所以它有个互斥器(mutex)作为并发控制之用。

```C%2B%2B
class PrettyMenu{
public:
    ...
    void changeBackground(std::istream& imgSrc);
    ...
private:
    Mutex mutex;               // 互斥器
    Image* bgImage;            // 目前的背景图像
    int imageChanges;          // 改变背景图像的次数
};
```

下面是 PrettyMenu 的 changeBackground 函数的一个可能实现：

```C%2B%2B
void PrettyMenu::changeBackground(std::istream& imgSrc){
    lock(&mutex);             // 取得互斥器
    delete bgImage;           // 删除旧的图像
    ++imageChanges;           // 修改图像变更次数
    bgImage = new Image(imgSrc);    // 安装新的背景图像
    unlock(&mutex);                 // 释放互斥器   
}
```

从 “异常安全性”的观点来看，这个函数很糟。“异常安全”有两个条件，而这个函数一个都不满足。

当异常抛出时，带有异常安全性的函数会：

- 不泄漏任何资源。上述代码一旦 "new Image(imgSrc)" 导致异常，对 unlock 的调用就不会执行，导致互斥器永远不会被释放。
- 不允许数据败坏。如果 "new Image(imgSrc)" 抛出异常，bgImage 就指向被删除的对象，imageChanges 也已经被累加，而其实并没有新的图像被成功安装起来。

#### 解决资源泄漏

解决资源泄漏的问题很容易，条款 13 讨论如何以对象管理资源，条款 14 也导入了 Lock class 作为一种 “确保互斥器被及时释放”的方法：

```C%2B%2B
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    Lock ml(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);
}
```



#### 解决数据败坏

此刻我们需要做个抉择，在能够做抉择之前，先学习一些术语：

- 基本承诺：如果异常被抛出，程序内的任何事物仍然保持在有效状态下。没有任何对象或数据结构会因此而败坏，所有对象都处于一种内部前后一致的状态（例如所有的 class 约束条件都继续获得满足）。例如：changeBackground 一旦异常了被抛出，PrettyMenu 对象可以继续拥有原背景图像，或者令它拥有某个缺省背景图像，但客户无法预知哪一种情况。
- 强烈保证：如果异常被抛出，程序状态不改变。调用这样的函数需要这样的认知：如果函数成功，就是完全成功，如果函数失败，程序会恢复到“调用函数之前的状态”。
- 不抛掷（nothrow）保证，承诺绝不抛出异常，因为它们总是能够完成它们原先承诺的功能。作用与内置类型（例如 int，指针等等）身上的所有操作都提供 nothrow 保证。这是异常安全码中一个必不可少的基础材料。



如果我们假设，函数带着“空白的异常明细”者必为 nothrow 函数，似乎合情合理，但其实不尽然。如下所示：

```C%2B%2B
int doSomething() throw();  // 空白的异常明细
```

这并不代表 doSomething 绝不会抛出异常，而是如果 doSomething 抛出异常，将是严重错误。实际上 doSomething 也许完全没有提供任何异常保证。函数的声明式（包括异常明细）并不能告诉你是否它是正确的，可移植的或高效的，也不能告诉你它是否提供任何异常安全保证。所有那些性质都由函数决定，无关乎声明。

为我们写的每一个函数提供一种安全保证。

#### 为 changeBackground 提供强烈保证

首先改变 PrettyMenu 的 bgImage 成员变量的类型，从一个 Image* 的内置指针改为 “用于资源管理”的智能指针。

第二，重新排列 changeBackground 内的语句次序，使得在更换图像之后才累加 imageChanges。一般这是个好策略：不要为了表示某件事发生而改变对象的状态，除非那件事真的发生了。

```C%2B%2B
class PrettyMenu {
    ...
    std::shared_ptr<Image>bgImage;
    ...
};

void PrettyMenu::changeBackground(std::istream& imgSrc){
    Lock ml(&mutex);
    bgImage.reset(new ImaSrc);   // 以 "new Image" 的执行结果设定 bgImage 内部指针
    ++imageChanges;
}
```

这里不需要再手动去 delete 旧图像，因为这个动作已经由智能指针内部处理调了，而且删除动作只发生在新图像被成功创建之后。更准确的说：std::shared_ptr::reset 函数只有在其参数被成功生成之后被调用，delete 只在 reset 函数内被使用。

美中不足是参数 imgSrc，如果 Image 构造函数抛出异常，有可能输入流（input stream）的读取记号（read marker）已被移走，而这样的搬移对程序其余部分是一种可见的状态改变。

所以 changeBackground 在解决这个问题之前只提供基本的安全保证。

#### 另一个方法 copy and swap

还有个一般化的设计策略会导致强烈保证，这个策略被称为 copy and swap。原则很简单，将打算修改的对象做出一份副本，然后在那副本身上做一切必要修改。若有任何修改动作抛出异常，原对象仍然保持不变。待所有改变都成功后，再将修改过的副本和原对象在一个不抛出异常的操作中置换（swap）。

通常将所有“隶属于对象的数据”从原对象放进另一个对象内，然后赋予原对对象一个指针，指向那个所谓的实现对象（implementation object）。这种手法称为 pimpl idiom。

```C%2B%2B
struct PMImpl{
    std::shared_ptr<Image>bgImage;
    int imageChanges;
}

class PrettyMenu{
    ...
private:
    Mutex mutex;
    std::shared_ptr<PMImpl> pImpl;
}

void PrettyMenu::changeBackground(std::istream& imgSrc){
    using std::swap;
    Lock ml(&mutex);
    std::shared_ptr<PMImpl>pNew(new PMImpl(*pImpl));
    pNew->bgImage.reset(new Image(imgSrc));   // 修改副本
    ++pNew->imageChanges;
    swap(pImpl,pNew);
}
```

为啥将 PMImpl 成为一个 struct 而不是一个 class，因为 PrettyMenu 的数据封装性已经由于“pImpl 是 private”而获得了保证。

"copy-and-swap" 策略是对对象状态做出 “全有或全无”改变的一个很好办法，但一般而言它并不保证整个函数有强烈的异常安全性：

```C%2B%2B
void someFunc(){
    ...             // 对 local 状态做一份副本
    f1();
    f2();
    ...             // 将修改后的状态置换过来 
}
```

很显然，如果 f1 或 f2 的异常安全性比 “强烈保证”低，就很难让 someFunc 成为“强烈异常安全”。



当写代码或修改代码时，思考如何让他具备异常安全性，首先是“以对象管理资源”，那可阻止资源泄漏，然后是挑选三个“异常安全保证”中的一个实施于我们写的每一个函数身上。应该选择合理条件下的最强烈等级，只有当你的代码调用了传统代码，才别无选择地将它设为“无任何保证”。将此决定写成文档，一来为函数用户着想，二来为将来的维护者着想。函数的“异常安全性”是其可见接口的一部分。



#### 总结

- 异常安全函数即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数分为三种可能的保证：基本型，强烈型，不抛异常型。
- “强烈保证”往往能够以 copy-and-swap 实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义。
- 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。



### 条款 30：透彻了解 inlining 的里里外外

inlining 函数原理：将对此函数的每一个调用都以函数本体替换。编译器最优化机制通常被设计用来浓缩那些“不含函数调用”的代码，当将一个函数 inline 之后，编译器或许有能力对函数本体执行语境进行优化。

inling 函数的优点是避免了函数调用产生的开销，但缺点是可能会大大增大目标码的大小，它可能会导致额外的换页行为，降低指令高速缓存装置的命中率，以及伴随这些而来的效率损失。

#### 隐喻声明 inline

记住，inline 只是对编译器的申请，不是强制命令。这项申请可以隐喻提出，也可以明确提出。隐喻方式是将函数定义于 class 定义式内：

```C%2B%2B
class Person {
public:
    ...
    int age() const { return theAge;}//一个隐喻的 inline 申请：age 被定义于 class 定义式内
private:
    int theAge;
};
```

friend 函数也可以被定义于 class 内，它们也是被隐喻声明为 inline

#### 显式声明 inline

显式声明 inline 的方法是在其定义式前加关键字 inline。例如标准的 max template 是这样实现的：

```C%2B%2B
template<typename T>                        
inline const T& std::max(const T& a, const T& b){
    return a<b?b:a;
}
```

大部分编译器拒绝将太过复杂（例如带有循环或递归）的函数 inlining。一个表面看似 inline 的函数是否真的 inline 取决于内置环境，主要取决于编译器。

#### outlined 函数本体 

有时候编译器愿意 inlining 某个函数，还是可能为该函数生成一个函数本体。例如，如果程序要取某个 inline 函数的地址，编译器通常为此函数生成一个 outlined 函数本体。

编译器通常不对“通过函数指针而进行的调用”实施inlining，这意味着对 inline 函数的调用有可能被 inlined，也可能 inlined，也可能不被inlined，取决于该调用的实施方式：

```C%2B%2B
inline void f() { ... }  // 假设编译器有意愿 inline “对 f 的调用”
void (*pf)() = f;  // pf 指向 f
...
f();                // 这个调用将被 inlined，因为它是一个正常调用
pf();               // 这个调用或许不被 inlined，因为它通过函数指针完成
```

inline 函数无法随着程序的升级而升级，如果 f 是程序库内的一个 inline 函数，客户将 "f 函数本体"编进程序中。一旦程序库设计者决定改变 f，所有用到 f 的客户端程序都必须重新编译。而如果 f 是 non-inline 函数，一旦它有任何修改，客户端只需重新连接就好。



平均而言：一个程序往往将  80% 的执行时间花费在 20% 的代码上，作为一个软件开发者，你的目标应该是找出这可以有效增进程序整体效率的 20% 代码，然后将它inline 或将他瘦身。

#### 总结

将大多数 inlining 限制在小型，被频繁调用的函数身上。这可以使日后的调试和二进制升级更容易，也可以使潜在的代码膨胀问题最小化。

不要只因为 function templates 出现在头文件，就将它们声明为 inline



### 条款 31：将文件间的编译依存关系降至最低

```C%2B%2B
#include <string>
#include "date.h"
#include "address.h"

class Person {
public:
    Person(const std::string& name, const Date& birth, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::string theName;       // 实现细目
    Date theBirthDate;         // 实现细目
    Address theAddress;        // 实现细目
}
```

#### 31.1 编译依存关系

上述代码存在一种编译依存关系（compilation dependency），如果上述头文件中的任何一个被改变，或这些头文件所依赖的其他头文件有任何改变，那么每一个含入 Person class 的文件就得重新编译，任何使用 Person class 的文件也必须重新编译。这样的连串编译依存关系会对许多项目造成麻烦。



#### 31.2 前置声明

定义一个类型时，编译器必须在编译期间知道对象的大小：

```C%2B%2B
int main(){
    int x;              // 定义一个 int
    Person p(params);   // 定义一个 Person
}
```

当编译器看到 x 的定义式，它必须知道分配多少内存，看到 p 也一样，它必须分配足够空间以放置一个 Person。编译器获得这些信息的唯一方法就是询问 class 的定义式。那么如果 class 定义式可以合法地不列出实现细目，编译器该如何分配多少空间？

对于 Java，Smalltalk 等语言这些问题并不存在，因为以那种语言分配对象，编译器只分配足够空间给一个指针（用以指向该对象）使用，也就是它们将上述代码视同这样子：

```C%2B%2B
int main(){
    int x;                // 定义一个 int
    Person* p;            // 定义一个指针指向 Person 对象
}
```

 这个也是合法的 C++ 代码，因此我们也可以“将对象的实现细目隐藏于一个指针背后”，针对 Person 我们可以这么做：把 Person 分割为两个 classes，一个只提供接口，另一个负责实现该接口。将那个负责实现的 class 称为 implementation class，取名为 PersonImpl，Person 定义如下：

```C%2B%2B
#include <string>
#include <memory>

class PersonImpl;   // Person 实现类的前置声明
class Date;         // Person 接口用到的 classes 的前置声明
class Address;

class Person {
public:
    Person(const std::string& name, const Date& birth, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::shared_ptr<PersonImpl>pImpl;      // 指针，指向实现物
}
```

 这样 Person 只含一个指针成员，指向其实现类（PersonImpl）。这种设计被称为 pimpl idiom（pimpl 是 "pointer to implementation" 的缩写）。

这样的设计下，Person 的客户端就完全与 Date，Address 以及 Person 的实现细目分离了。那些 classes 的任何实现修改都不需要 Person 客户端重新编译。

此外，由与客户无法看到 Person 的实现细目，也就不可能写出“取决于那些细目”的代码，这就是“接口与实现分离”。



这个分离的关键是以“声明的依存性”替换“定义的依存性”，那就是编译依存性最小化的本质：现实中让头文件尽可能自我满足，万一做不到，则让他与其他文件内的声明式（而非定义式）相依。

- 如果使用 object references 或 object pointers 可以完成任务，就不要使用 objects。因为只靠一个类型声明就可以定义指向该类型的 references 和 pointers。但如果定义某类型的 objects，就需要用到该类型的定义式。
- 尽量以 class 声明式替换 class 定义式。注意：当你声明一个函数而它用到某个 class 时，你并不需要该 class 的定义，即使函数以 by value 方式传递该类型的参数（或返回值）亦然：

```C%2B%2B
class Date;           // class 声明式
Date today();         // 没问题
void clearAppointments(Date d);   // Date 定义式
```

声明 today 函数和 clearAppointments 函数而无需定义 Date，因为一旦任何人调用这些函数，调用之前 Date 定义式一定得先曝光才行。假设一个函数库包含数百个函数声明，不太可能每个客户调用每一个函数，如果能够将 “提供 class 的定义式”（通过 #include）的义务从“函数声明所在”的头文件转移到“函数被调用”的客户端文件，就可以将“并非真正必要的类型定义”与客户端之间的编译依存性取除掉。例如：

- 为声明式和定义式提供不同的头文件。为了遵守上述准则，需要两个头文件，一个用于声明，一个用于定义式。如果某个声明式文件被改变了，两个文件都得改变。因此程序库客户端应该总是 #include 一个声明文件而非前置声明若干函数。

```C%2B%2B
#include "datafwd.h"        // 这个头文件内声明（但未定义） class Date
Date today();
void clearAppointments(Date d);   // Date 定义式
```

只含声明式的头文件名为  "datafwd.h" ，命名方式采用 C++ 标准程序库头文件的 <iosfwd>， <iosfwd> 内含 iostream 各组件的声明式，其对应定义则分布在若干不同的头文件内，包括 <sstream> <streambuf> <fstream> 和 <iostream>



#### 31.3 export

C++ 提供关键字 export，允许将 template 声明式和 template 定义式分割于不同的文件内。但有些编译器可能并不支持这个关键字。



#### 31.4 Handle class

像 Person 这样使用 pimpl idiom 的classes，往往被称为 Handle classes，将它们的所有函数转交给相应的实现类并由后者完成实际工作。例如下面是 Person 两个成员函数的实现：

```C%2B%2B
// 我们正在实现 Person class，所以必须 #include 其定义式
#include "Person.h"    
 
// 必须#include PersonImpl 的 class 定义式，否则无法调用其成员函数
// PersonImpl 有着和 Person 完全相同的成员函数，两者接口完全相同 
#include "PersonImpl.h" 

Person::Person(const std::string& name, const Date& birth, const Address& addr)
    :pImpl(new PersonImpl(name,birth,addr)){}
    
std::string Person::name() const{
    return pImpl->name();
}    
                            
                            
```

让 Person 变成一个 Handle class 并不会改变它做的事，只会改变它做事的方法。

#### 31.5  Interface class

另一个制作 Handle class 的办法是，令 Person 成为一种特殊的 abstract base class（抽象基类），称为 Interface class。这种 class 的目的是详细一一描述 derived class 的接口，因此它通常不带成员变量，也没有构造函数，只有一个 virtual 析构函数，以及一组 pure virtual 函数，用来描述整个接口。

一个针对 Person 而写的 Interface class 获取看起来像这样：

```C%2B%2B
class Person {
public:
    virtual ~Person();
    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0
    virtual std::string address() const = 0;
    ...
};
```

这个 class 的客户必须以 Person 的pointers 或 reference 来编写应用程序，因为它不可能针对“内含 pure virtual 函数”的 Person classes 具现出实体（然而却可能对派生自 Person 的 classes 具现出实体）。就像 Handle classes 的客户一样，除非 Interface class 的接口被修改否则其客户不需要重新编译。

Interface class 的客户必须有办法为这种 class 创建对象，它们通常调用一个特殊函数，用来构造那个 derived class 对象，这样的函数称为工厂函数或 virtual 构造函数，它们返回指针（更可取的智能指针）指向动态分配的对象，而该对象支持 Interface class 的接口，这样的函数往往在 Interface class 内被声明为 static

```C%2B%2B
class Person {
public:
    virtual ~Person();
    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0;
    virtual std::string address() const = 0;

    static std::shared_ptr<Person> create(const std::string& name, const Date& brithday, const Address& addr){
        return std::shared_ptr<Person> (new RealPerson(name,brithday,addr));
    }

};
```

客户会这样使用它们：

```C%2B%2B
std::string name;
Date dateofBirth;
Address address;
...
// 创建一个对象，支持 Person 接口
std::shared_ptr<Person> pp(Person::create(name,dataofBirth,address));
...
// 通过 Person 的接口来使用这个对象
std::cout << pp->name() << " " << pp->birthDate()<< " " << pp->address();

// 当 pp 离开作用域，对象会被自动删除
 
```

当然，支持 Interface class 的那个具象类(concrete classes)必须被定义出来，而且真正的构造函数必须被调用。一切都在 virtual 构造函数实现代码所在文件内发生。假设 Interface class Person 有个具象类 deriveded class RealPerson，后者提供继承而来的 virtual 函数的实现

```C%2B%2B
class RealPerson: public Person {
public:
    RealPerson(const std::string& name, const Date& brithday, const Address& addr);


    virtual ~RealPerson() {}
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;


private:
    std::string theName;       // 实现细目
    Date theBirthDate;         // 实现细目
    Address theAddress;        // 实现细目
};
```

有了 RealPerson 之后，写出 Person::create 就很容易了：

```C%2B%2B
static std::shared_ptr<Person> Person::create(const std::string& name, const Date& brithday, const Address& addr){
        return std::shared_ptr<Person> (new RealPerson(name,brithday,addr));
    }
```

一个更现实的 Person::create 实现代码会创建不同类型的 derived class 对象，取决于诸如额外的参数值，然后实现接口所覆盖函数。



Handle classes 和 Interface classes 解除了接口和实现之间的耦合关系，从而降低了文件间的编译依存性。但代价就是：运行期间丧失若干速度，为每个对象超额付出若干内存。

Handle classes的缺点：

- 速度方面——成员函数必须通过 implementation pointer 取得对象数据，那会为每一次访问增加一层间接性。implementation pointer 必须被初始化，指向一个动态分配得来的 implementation object，又产生了动态内存分配和释放的开销
- 空间方面——每一个对象消耗的内存必须增加 implementation pointer 的大小
- 异常：动态内存分配因内存不足，而遭遇 bad_alloc 的可能性



Interface classes 的缺点：

- 速度方面——每个函数都是 virtual ，必须为每个函数调用付出一个间接跳跃成本（查询虚函数表）
- 空间方面——Interface class 派生的对象必须包含一个 vptr，这个指针可能会增加存放对象所需的内存数量——实际取居于这个对象除了 Interface class 之外是否还有其他 virtual 函数来源。



最后，无论是 Handle classes 还是 Interface classes，一旦脱离 inline 函数都无法有太大作为。条款 30 解释了为什么函数本体为了被 inlined 必须置于头文件内，但 Handle classes 和 Interface classes 正是特别被设计用来隐藏实现细节如函数本体。 



#### 31.6 总结

- 支持“编译依存性最小化”的一般构想：相依于声明式，不要相依于定义式，基于此构想的两个手段是 Handle classes 和 Interface classes。
- 程序库头文件应该以“完全且仅有声明式”的形式存在，这种做法无论是否涉及 templates 都适用。



## 六、继承与面向对象设计

C++ 的面向对象编程（OOP）具有以下特点：

- 支持单一继承或多重继承
- 继承类型可以是：public, protected 或 private，也可以是 virtual
- 成员函数类型有：virtual, non-virtual, pure virtual
- virtual 函数意味着“接口必须被继承”
- non-virtual 函数意味着“接口和实现都必须被继承”



### 条款 32：确定你的 public 继承塑模出 is-a 关系

#### 32.1 public 继承

一个 class D 以 public 形式继承 class B，意味着每一个类型为 D 的对象同时也是一个类型为 B 的对象，反之不成立。B 表现出比 D 更一般化的概念，D 比 B 表现出更特殊的概念。



```C%2B%2B
class Person { ... };
class Student: public Person { ... }; 
```

根据常识，每个学生都是人，但并非每个人都是学生。这就是 public 继承体系的主张。



于是，在 C++ 领域中，任何函数如果期望获得一个类型为 Person（pointer-to-Person 或 reference-to-Person） 的实参，都愿意接受一个 Student 对象（pointer-to-Student 或 reference-to-Student）。

```C%2B%2B
void eat(const Person& p);      // 任何人都会吃
void study(const Student& s);   // 只有学生才到校学习
Person p;
Student s;
eat(p);    // 没问题，p 是人
eat(s);    // 没问题，s 是学生，而学生也是（is-a）人
study(s);  // 没问题，s 是个学生
study(p);  // 错误，p 不是个学生
```

#### 32.2  is-a 关系

is-a 关系，翻译为中文表示“是一个/是一种”关系。例如前面的 Student 和 Person 的例子，一个学生也“是一个”人。这就是所谓的 is-a 关系。

public 继承和 is-a 之间的等价关系听起来简单，但也需要注意某些违反直觉的问题。例如：

##### 企鹅和鸟

企鹅(penguin) 是一种鸟，鸟可以飞，这些都是事实。观察以下继承关系：

```C%2B%2B
class Bird{
public:
    virtual void fly();       // 鸟可以飞
};

class Penguin: public Bird{   // 企鹅是一种鸟
    ...
};
```

这个继承体系表明企鹅可以飞，根据常识，企鹅不会飞，很明显这个继承关系有问题。



如果思想再严谨一些，我们知道，不是所有鸟都会飞。现在我们设计以下继承关系：

```C%2B%2B
class Bird {
    ...             // 没有声明 fly 函数
};

class FlyingBird: public Bird {
public:
    virtual void fly();
    ...
};

class Penguin: public Bird {
    ...           // 没有声明 fly 函数
};
```

这样的继承体系比原先的设计更能反映我们真实的意思。



即便如此，我们仍然未能完全处理好这些鸟事，因为对某些系统而言，可能不需要区分会飞的鸟和不会飞的鸟。如果我们的程序完全不在乎飞行，那么原先的“双 classes 继承体系”就令人满足了。这反应了一个事实，世界上并不存在一个“适用于所有软件”的完美设计。所谓的最佳设计，取决于系统希望做什么事，包括现在与未来。



另一种办法是为企鹅重新定义 fly 函数，令它产生一个运行期错误：

```C%2B%2B
void error(const std::string& msg);    // 定义于另外某处
class Penguin: public Bird {
public:
    virtual void fly() { error("试图让一只企鹅飞"); }
};
```

这是一种在运行期侦测错误的办法。

除此之外，我们还可以在编译期间就可以侦测出错误用法，如下所示：

```C%2B%2B
class Bird{
    ...                // 没有声明 fly 函数
};

class Penguin: public Bird {
    ...                // 没有声明 fly 函数
};

// 现在，如果企图让企鹅飞，编译器会发出警告
Penguin p;
p.fly();     // 错误
```

#### 32.3 总结

"public 继承"意味着 is-a。适用于 base classes 身上的每一件事情一定也适用于 derived classes 身上，因为每一个 derived class 对象也都是一个 base class 对象。



### 条款 33：避免遮掩继承而来的名称

#### 33.1 作用域与查找规则

本条款所讲的内容主要和作用域有关。例如以下代码：

```C%2B%2B
int x;             // global 变量
void someFunc(){
    double x;      // local 变量
    std::cin >> x; // 读取一个新值赋予 local 变量 x
}
```

读取数据的语句，使用的是 local 变量 x，而不是 global 变量 x，因为内层作用域的名称会遮盖外围作用域的名称。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MDA2ZDM1MmJkNGE1MmVhMmYxZDg4MzlmMTk0ZDczNDNfZFZaNm5ueFQ4QjBOQUdsSk8ybllGQVdIZ2J2ZzlzMzVfVG9rZW46Ym94azQ1dGVFU2FsWTdxQXlQOGtDaEpYeE5oXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

当涉及到继承时，derived class 作用域被嵌套在 base class 作用域内，像这样：

```C%2B%2B
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf2();
    void mf3();
    ...
};

class Derived: public Base {
public:
    virtual void mf1();
    void mf4();
    ...
};
```

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NGRjNWQ3NzUzOGI1NGMwMWRmZWQ2N2VmNjY2MWE4MTlfVjAzRVFDcXFaeG1NQnkyUDdSemxSd0VLZXFIeFg5TlpfVG9rZW46Ym94azRRQkxVeVVPUmdGVzVWeE9rNFdFM3lnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述例子中混合了 public 和 private 名称，成员函数包括 pure virtual, virtual, non-virtual 三种，只是为了强调接下来我们讨论的只和名称有关，和其它无关。

假设 derived class 内的 mf4 的实现代码部分像这样：

```C%2B%2B
void Derived::mf4(){
    ...
    mf2();
    ...
}
```

当编译器看到名称 mf2，它会查找各个作用域。

1. 编译器首先查找 local 作用域（也就是 mf4 覆盖的作用域）
2. 在那没找到任何名为 mf2，于是查找外围作用域，也就是 class Derived 覆盖的作用域
3. 如果还是没找到任何东西名为 mf2，就再往外围移动，本例为 base class，在那编译器找到一个名为 mf2 的东西，于是就停止查找。

如果 Base 内还是没有 mf2，就查找 Base 所在的  namespace（如果有的话），最后查找 global 作用域。



现在我们重载 mf1 和 mf3，并且添加一个新版 mf3 到 Derived 中。

```C%2B%2B
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
    ...
};

class Derived:public Base{
public:
    virtual void mf1();
    void mf3();
    void mf4();
    ...
};
```

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YzFkMTkwYjJlMzE4ODU4ZTZjNjcxYzIzNDU1Mjc1ZTZfUVpsV3RyeHdHWXphQ2R2ZWNIOWtUMjdxZUpnUGs0ZmpfVG9rZW46Ym94azQ3WFhrV21jZ21ZSnlJUEhweWJZUm9mXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

Base class 内所有名为 mf1 和 mf3 的函数都被 derived class 内的 mf1 和 mf3 函数遮掩掉了。从名称查找观点来看，Base::mf1 和 Base::mf3 不再被 Derived 继承

```C%2B%2B
Derived d;
int x;
...
d.mf1();             // 没问题，调用 Derived::mf1
d.mf1(x);            // 错误，因为 Derived::mf1 遮掩了 Base::mf1
d.mf2();             // 没问题，调用 Base::mf2
d.mf3();             // 没问题，调用 Derived::mf3
d.mf3(x);            // 错误，因为 Derived::mf3 遮掩了 Base::mf3
```

这些行为背后的理由是为了防止你在程序库或应用框架内建立新的 derived class 时附带地从疏远的 base class 继承重载函数。

#### 33.2 using 声明式

不幸的是我们通常会想使用被遮掩的名称，可以这么做：

```C%2B%2B
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
    ...
};

class Derived:public Base{
public:
    using Base::mf1;       // 让 Base class 内名为 mf1 和 mf3 的所有东西
    using Base::mf3;       // 在 Derived 作用域内可见
    virtual void mf1();    
    void mf3();
    void mf4();
    ...
};
```

现在:

```C%2B%2B
Derived d;
int x;
...
d.mf1();             // 没问题，调用 Derived::mf1
d.mf1(x);            // 没问题，调用 Base::mf1
d.mf2();             // 没问题，调用 Base::mf2
d.mf3();             // 没问题，调用 Derived::mf3
d.mf3(x);            // 没问题，调用 Base::mf3
```

如果想使用被 Derived 遮掩的 Base 名称，可以使用 using 声明式，该 using 声明式可被继承。



#### 32.3 转交函数

加入在 private 继承下，如果我们只想继承某个被遮掩的函数，而不是以某个名字命名的所有函数，这个时候就不能使用 using。

```C%2B%2B
class Base {
public:
    virtual void mf1();
    virtual void mf1(int);
    ...             // 与前相同
};

class Derived::private Base {
public:
    virutal void mf1(){    // 转交函数
        Base::mf1();       // 暗自成为 inline
    }
    ...
};
...
Derived d;
int x;
d.mf1();              // 很好，调用了 Derived::mf1
d.mf1(x);             // 错误！Base::mf1 被遮掩了 
```

inline 转交函数的另一个用途是为那些不支持 using 声明式的老旧编译器提供访问被遮掩名称的方法。

#### 32.4 总结

derived classes 内的名称会遮掩 base classes 内的名称。

为了让被遮掩的名称再次被使用，可以使用 using 声明式或转交函数。



### 条款 34：区分接口继承和实现继承

C++ 的 public 继承分为：

- 函数接口(function interfaces)继承
- 函数实现(function implementations)继承

现通过以下继承关系举例：

```C%2B%2B
class Shape{
public:
    virtual void draw() const = 0;
    virtual void error(const std::string& msg);
    int objectID() const;
    ...
};

class Rectangle: public Shape { ... };
class Ellipse: public Shape { ... };
```

Shape 强烈影响所有以 public 形式继承它的 derived classes，因为以下方面：

- 成员函数的接口总是被继承

如条款 32 所说，public 继承意味着 is-a（是一种）。对 base class 为真的任何事情一定也对其 derived classes 为真。如果某个函数可以施行于某 class 身上，一定也可施行于其 derived classes 身上。

Shape 声明了三种类型的成员函数，分别是 pure virtual 函数，virtual 函数，non-virtual 函数。这些不同的声明分别表示什么呢？

首先考虑 pure virtual 函数 draw:

```C%2B%2B
class Shape{
public:
    virtual void draw() const = 0;
    ...
};
```

该 pure virtual 函数具有两个突出特性：

- 它们必须被任何“继承了它们”的具象 class 重新声明。
- 它们在抽象 class 中通常没有定义。

把上述两个性质放在一起，我们就知道：

#### 34.1 声明一个 pure virtual 函数的目的是为了让 derived classes 只继承函数的接口

- Shape::draw 的声明式好像在对具象 derived classes 设计者说：“你必须提供一个 draw 函数，但我不干涉你怎么实现它。”
- 令人意外的一点是，我们可以为 pure virtual 函数提供定义，也就是可以为 Shape::draw 供应一份实现代码，但调用它的唯一途径是“调用时明确指出其 class 名称”：

```C%2B%2B
Shape* ps = new Shape;       // 错误！Shape 是抽象的
Shape* ps1 = new Rectangle;  // 没问题
ps1->draw();                 // 调用 Rectangle::draw
Shape* ps2 = new Ellipse;    // 没问题 
ps2->draw();                 // 调用 Ellipse::draw
ps1->Shape::draw();          // 调用 Shape::draw
ps2->shape::draw();          // 调用 Shape::draw
```

为 pure virtual 函数提供定义用途有限，但接下来你会看到，它可以实现一种机制，为 (非纯) impure virtual 函数提供更平常更安全的缺省实现。

Base class 定义一个 virtual 函数，Derived class 可以选择实现该虚函数，也可以选择不实现该虚函数，当选择不实现它时，Base class 的相应 virutal 函数作为一个缺省实现版本。

#### 34.2 声明简朴的（非纯）impure virtual 函数的目的，是让 derived classes 继承该函数的接口和缺省实现

考虑 Shape::error 这个例子：

```C%2B%2B
class Shape {
public:
    virtual void error(const std::string& msg);
    ...
};
```

该接口表示，每个 class 都必须支持一个“当遇上错误时可调用”的函数，但每个 class 可自由处理错误。如果某个 class 不想针对错误做出任何特殊行为，它可以退回到 Shape class 提供的缺省错误处理行为。

Shape::error 的声明式告诉 Derived classes 的设计者：“你必须支持一个 error 函数，但如果你不想自己写一个，可以使用 Shape class 提供的缺省版本”。



但是，允许 impure virtual 函数同时指定函数声明和函数缺省行为，有可能造成危险。为了探讨原因，下以 XYZ 航空公司的飞机继承体系举例，该公司只有 A 型和 B 型两种飞机，两者都以相同方式飞行。因此 XYZ 设计出这样的继承体系：

```C%2B%2B
class Airport { ... };  // 用以表示机场
class Airplane {
public:
    virtual void fly(const Airport& destination);
    ...
};

void Airplane::fly(const Airport& destination){
    缺省代码，将飞机飞至指定的目的地
}

class ModelA: public Airplane { ... };
class ModelB: public Airplane { ...};
```

因为原则上“不同型号飞机需要不同的 fly 实现”，Airplane 被声明为 virtual，然而为了避免在 ModelA 和 ModelB 中撰写相同代码，缺省飞行行为由 Airplane::fly 提供，它同时被 ModelA 和 ModelB 继承。

现在假设 XYZ 决定购买一种新式 C 型飞机，C 型和 A 型及 B 型的飞行方式不同。XYZ 公司的程序员在继承体系中针对 C 型飞机添加了一个 class，但由于他们着急让新飞机上线服务，竟忘了重新定义 fly 函数：

```C%2B%2B
class ModelC: public Airplane{
    ...                   // 未声明 fly 函数
};

// 然后在代码中有一些诸如此类的动作
Airport PDX(...);      
Airplane* pa = new ModelC;
...
pa->fly(PDX);          // 调用 Airplane::fly
```

这样将酿成大灾难，这个程序试图以 ModelA 或 ModelB 的飞行方式来飞 ModelC。

问题不在 Airplane::fly 有缺省行为，而在于 ModelC 未重写该虚函数，而是继承了该缺省行为。幸运的是我们可以轻易做到“提供缺省实现给 derived classes，但除非它们明白要求否则面谈”，此方法在于切断“virtual 函数接口”和其“缺省实现”之间的连接。下面是一种做法：

```C%2B%2B
class Airplane {
public:
    virtual void fly(const Airport& destination) = 0;
    ...
protected：
    void defaultFly(const Airport& destination);
};

void Airplane::defaultFly(const Airport& destination){
    缺省行为，将飞机飞至指定的目的地
}
```

Airplane::fly 已被改为一个 pure virtual 函数，只提供飞行接口，其缺省行为只出现在 Airplane class 中。

```C%2B%2B
class ModelA: public Airplane { 
public:
    virtual void fly(const Airport& destination){
        defaultFly(destination);
    }
    ...
};
class ModelB: public Airplane { 
public:
    virtual void fly(const Airport& destination){
        defaultFly(destination);
    }
    ...
};
```

现在 ModelC 不可能意外继承不正确的 fly 实现代码：

```C%2B%2B
class ModelC: public Airplane{
public:
    virtual void fly(const Airport& destination);
    ...
};

void ModelC::fly(const Airport& destination){
    将 C 型飞机飞至目的地
}
```

注意 Airplane::defaultFly 现在成了 protected，因为它是 Airplane 及其derived classes 的实现细目。乘客应该只在意飞机能不能飞，不在意它们怎么飞。



有些人也反对以不同的函数分别提供接口和缺省实现，他们担心因过度雷同的函数名称而引起的 class 命名空间污染问题，但他们也同意，接口和缺省实现应该分开。这个矛盾应该如何解决呢？我们可以利用“pure virtual 函数必须在 derived classes 中重新声明，但他们也可以拥有自己的实现”这一事实。

```C%2B%2B
class Airplane {
public:
    virtual void fly(const Airport& destination) = 0;
    ...
};

void Airplane::fly(const Airport& destination)   // pure virtual 函数实现
{
    缺省行为，将飞机飞至目的地
}

class ModelA: public Airplane {
public:
    virtual void fly(const Airport& destination){
        Airplane::fly(destination);
    }
};

class ModelB: public Airplane {
public:
    virtual void fly(const Airport& destination){
        Airplane::fly(destination);
    }
};

class ModelC : public Airplane {
public:
    virtual void fly(const Airport& destination);
    ...
};

void ModelC::fly(const Airport& destination){
    将 C 型飞机飞至指定的目的地
}
```



#### 34.3 声明 non-virtual 函数的目的是为了令 derived classes 继承函数的接口及一份强制性实现

最后，讨论 Sharp 的 non-virtual 函数 objectID:

```C%2B%2B
class Shape {
public:
    int objectID() const;
    ...
};
```

如果成员函数是 non-virtual 函数，意味着它并不打算在 derived classes 中有不同的行为。

可以把 Sharpe::objectID 的声明看作是：“每个 Sharp 对象都有一个用来产生对象识别码的函数；此识别码总是采用相同计算方法，该方法由 Sharpe::objectID 的定义式决定，任何 derived class 都不应该尝试改变其行为”。



#### 34.4 总结

接口继承和实现继承不同。在 public 继承下，derived classes 总是继承 base class 的接口

pure virtual 函数只具体指定接口继承

简朴的（非纯）impure virtual 函数具体指定接口继承及缺省实现继承。

non-virtual 函数具体指定接口继承以及强制性实现继承。



### 条款 35：考虑 virtual 函数以外的其他选择

现在为一个游戏内的人物设计一个继承体系，现提供一个成员函数 healthValue，它返回一个整数，表示人物的健康程度。不同的人物有不同的方式计算他们的健康程度，将 healthValue 声明为 virtual 是很常见的做法。

```C%2B%2B
class GameCharacter {
public:
    virtual int healthValue() const;    // 返回人物的健康指数
    ...                                 // derived classes 可重新定义它
}
```

为将 healthValue 声明为 pure virtual，是想为其保留一个默认实现。从某些角度来看上述设计存在某些弱点，现在考虑一些其他解法。

#### 35.1 由 Non-Virtual Interface 手法实现 Template Method 模式（模板方法模式）

关于模板方法模式可以参考：https://refactoringguru.cn/design-patterns/template-method

所谓 Template Method 模式，当多个派生类存在结构相似的代码时，可以将共同的代码提取到超类中，将这些代码比喻为一个大型算法，不同的派生类可以重写这个大型算法的一部分。如下所示：

```C%2B%2B
class GameCharacter {
public:
    int healthValue() const                    // derived classes 不重新定义它
    {
        ...                                    // 做一些事前工作
        int retVal = doHealthValue();     
        ...                                    // 做一些事后工作
        return retVal;
    }
private:
    virtual int doHealthValue() const          // derived classes 可重新定义它
    {
        ...                                    // 缺省算法，计算健康指数
    }
}
```

像上述这样令 “客户通过 public non-virtual 成员函数间接调用 private virtual 函数”称为 non-virtual interface（NVI）手法。我们把这个 non-virtual 函数(healthValue) 称为 virtual 函数的外覆器(wrapper)。

这个手法的优点：

- 调用 doHealthValue 之前可以做一些事前工作：锁定互斥器，日志记录，验证 class 的约束条件，验证函数先诀条件
- 调用 doHealthValue 之后做一些事后工作：解除互斥器锁定，验证函数的事后条件，再次验证 class 的约束条件等等。

在 NVI 手法下 virtual 函数不一定得是 private。某些 class 继承体系要求 derived class 在 virtual 函数的实现内必须调用其  base class 的某些属性或方法，virtual 函数就必须是 protected。



#### 35.2 由  std::function 完成 Strategy 模式

##### 函数指针法

另一个主张是：“人物健康指数与人物类型无关”，那这样的计算就不需要“人物”这个 成分，例如我们可能要求每个人物的构造函数接受一个指针，指向一个健康计算函数，可以调用该函数进行实际的计算：

```C%2B%2B
class GameCharacter;           // 前置声明
//以下函数是计算健康指数的缺省算法
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf)
    {}
    int healthValue() const;
    { return healthFunc(*this); }
private:
    HealthCalcFunc healthFunc;
};
```

这个做法是常见的 Strategy 设计模式的简单应用，相比使用 virtual 函数，它提供了某些有趣弹性：

- 同一人物类型的不同实体可以有不同的健康计算函数：

```C%2B%2B
class EvilBadGuy: public GameCharacter {
public:
    explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc):GameCharacter(hcf)
    { ... }
    ...
};
int loseHealthQuickly(const GameCharacter&);    // 健康指数计算函数1
int loseHealthSlowly(const GameCharacter&);     // 健康指数计算函数2
EvilBadGuy edg1(loseHealthQuickly);
EvilBadGuy edg2(loseHealtySlowly);              // 相同类型的人物搭配不同的健康计算方式
```

- 某已知人物的健康计算函数可以在运行期变更。例如 GameCharacter 可提供一个成员函数 setHealthCalculator，用来替代当前的健康指数计算函数



##### std::function 对象

基于函数指针的做法有点死板，为什么一定是个函数指针，而不是函数对象；为不是不能是个成员函数。

现在我们不再使用函数指针，而是改用一个类型为 std::function 的对象，这样的对象可持有任何可调用物（函数指针，函数对象，成员函数指针），例如：

```C%2B%2B
class GameCharacter;           // 前置声明
//以下函数是计算健康指数的缺省算法
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    typedef std::function<int (const GameCharacter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf)
    {}
    int healthValue() const;
    { return healthFunc(*this); }
private:
    HealthCalcFunc healthFunc;
};
```

和前一个设计几乎相同，唯一的不同是 GameCharacter 持有一个 std::function 对象，相当于一个指向函数的泛化指针，它可以提供更多的弹性：

```C%2B%2B
short calcHealth(const GameCharacter&);   // 健康计算函数，注意其返回类型是 short

struct HealthCalculator{                  // 为计算健康而设计的函数对象
    int operator()(const GameCharacter&) const
    { ... }
};

class GameLevel {
public：
    float health(const GameCharacter&) const; // 成员函数，用以计算健康
    ... 
};

class EvilBadGuy: public GameCharacter {
public:
    explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc):GameCharacter(hcf)
    { ... }
    ...
};

class EyeCandyCharacter: public GameCharacter { // 另一个人物类型，假设构造函数与 EvilBadGuy 同

    ...
};

EvilBadGuy ebg1(calcHealth);    // 人物1，使用某个函数计算健康指数
EyeCandyCharacter eccl(HealthCalculator());   // 人物 2，使用某个函数对象

GameLevel currentLevel;
EvilBadGuy egb2(
    std::bind(&GameLevel::health, &currentLevel, _1) 
); // 人物3，使用某个成员函数
```

GameLevel::health 接受两个参数，一个是  this 指针，绑定到 &currentLevel，另一个是 const GameCharacter&，_1 表示占位符，即调用 healthFunc 传入的第一个参数。

#### 35.3 古典的 Strategy 模式

古典的 Strategy 做法会把健康计算函数做成一个分离的继承体系中的 virtual 成员函数，如下 UML 所示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=OTdkMjQxYWMxZmU2MDE1Zjg5NjMyOTYzZDExMmNkZTBfUEdQN3JHdWd4WEJid1RiUEVhQlNDYlFLVmNGNGYzNEtfVG9rZW46Ym94azRCdkQ0c0RLcHVlS3lBTG1zTlFpS3FnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

对应的代码如下：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NWEzNzI1ZDYyNmRmYzM1MDQ1OGQ2YTU0N2RmOGJkZGRfSlBJeFZKOXJlSGpNUlNmY0VQOFpYZWh0RnpNTlNVcFpfVG9rZW46Ym94azRUWjVMUmFOUTZwQzc5WEdETkJPakdlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

本条款的根本忠告是，当为解决问题而寻找某个设计方法时，不妨考虑 virtual 函数的替代方案：

- 使用 non-virtual interface（NVI）手法，那是 Template Method 设计模式的一种特殊形式，
- 将 virtual 函数替换为 “函数指针成员变量”，这是 Strategy 设计模式的一种。
- 以 std::function 成员变量替换 virtual 函数，这也是 Strategy 设计模式的传统实现手法。
- 将继承体系内的 virtual 函数替换为另一个继承体系内的 virtual 函数，这是 Strategy 设计模式的传统实现手法。



### 条款 36：绝不重新定义继承而来的 non-virtual 函数

观察以下例子：

```C%2B%2B
class B {
public:
    void mf();
    ...
};

class D: public B { ... }
```

先定义一个类型为 D 的对象 ``D x;``

现有以下行为：

```C%2B%2B
B* pB = &x;         // 获得一个指针指向 x
pB->mf();           // 经指针调用 mf
```

异于以下行为：

```C%2B%2B
D* pD = &x; 
pD->mf();
```

按照正常的逻辑，两者都通过对象 x 调用成员函数 mf，所以他们的行为应该相同，是这样吗？

但是，如果 mf 是个 non-virtual 函数，而 D 定义有自己的 mf 版本，两次调用的行为就不一样了：

```C%2B%2B
class D: public B {
public:
    void mf();      // 遮掩了 B::mf
    ...
};

pB->mf();           // 调用 B::mf
pD->mf();           // 调用 D::mf
```

造成此行为的原因是，non-virtual 函数，如 B::mf 和 D::mf 都是静态绑定，意思是，由于 pB 被声明为 pointer-to-B，通过 pB 调用的 non-virtual 函数永远是 B 所定义的版本，即使 pB 指向一个类型为“B 的派生对象”。

另一方面，virtual 函数却是动态绑定，如果 mf 是个 virtual 函数，无论是通过 pB 或 pD 调用 mf，都会导致调用 D::mf，因为 pB 和 pD 真正指的都是一个类型为 D 的对象



所谓的 public 继承意味 is-a（是一种）关系，对于 B 和 D 以及 non-virtual 成员函数 B::mf ：

- 适用于 B 对象的每一件事，也适用于 D 对象，因为每个 D 对象都是一个 B 对象
- B 的 derived classes 一定会继承 mf 的接口和实现，因为 mf 是 B 的一个 non-virtual 函数

现在如果 D 重新定义 mf，上述设计就会出现矛盾。

总之，任何情况下都不该重新定义一个继承而来的 non-virtual 函数。



### 条款 37：绝不重新定义继承而来的缺省参数值

本条款成立的理由是：virtual 函数是动态绑定，而缺省参数值却是静态绑定。

#### 37.1 静态绑定

对象的所谓静态，就是它在程序中被声明时所采用的类型。考虑以下继承体系：

```C%2B%2B
// 一个用以描述几何形状的 class
class Shape {
public:
    enum ShapeColor {Red, Green, Blue };
    // 所有形状都必须提供一个函数，用来绘出自己
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};

class Rectangle: public Shape {
public:
    // 注意，赋予了不同的参数值，这很糟糕
    virtual void draw(ShapeColor color = Green) const;
    ...
};

class Circle: public Shape{
public:
    virtual void draw(ShapeColor color) const;
};
```

这个继承体系图如下：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MzVmZjE1YmRhNjQzZmVlYzlhMTg3NDg4Y2Y3Y2M3ZWVfZVo2dzh1V05hYndBN2pmRzhud1ByUjc5VnBPTTBiTnNfVG9rZW46Ym94azR6N2ZPQUx6RHVKZHY5OVI3bkVmTDBlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

现在考虑这些指针：

```C%2B%2B
Shape* ps;                 // 静态类型为 Shape*
Shape* pc = new Circle;    // 静态类型为 Shape*
Shape* pr = new Rectangle; // 静态类型为 Shape*
```

上述的 ps, pc, pr 都被声明为 pointer-to-Shape 类型，不论他们真正指向什么，它们的静态类型都是 Shape*

 

#### 37.2 动态绑定

所谓的动态类型是指：“目前所指向对象的类型”，也就是说，动态类型可以表现出一个对象将会有什么行为。就上述例子，pc 的动态类型为 Circle*，pr 的动态类型为 Rectangle*，ps 没有动态类型，因为它尚未指向任何对象。

动态类型可以在程序执行过程中改变（通常经由赋值动作）：

ps = pc;       //  ps 的动态类型是 Circle*

ps = pr;       //  ps 的动态类型是 Rectangle*

virtual 函数是动态绑定而来，调用一个 virtual 函数时，究竟调用哪一份函数实现代码，取决于发出调用的那个对象的动态类型：

```C%2B%2B
pc->draw(Shape::Red);        // 调用 Circle::draw(Shape::Red)
pr->draw(Shape::Red);        // 调用 Rectangle::draw(Shape::Red)     
```



#### 37.3 带有缺省参数值的 virtual 函数

virtual 函数是动态绑定，而缺省参数值却是动态绑定。意思是可能会在“调用一个定义于 derived class 内的virtual 函数”的同时，却使用 base class 为它指定的缺省参数值：

pr->draw();            // 调用 Rectangle::draw(Shape::Red)!

此例中，pr 的动态类型是 Rectangle*，所以调用的是 Rectangle 的 virtual 函数。Rectangle::draw 函数的缺省参数值应该是 GREEN，但由于 pr 的静态类型是 Shape*，所以此调用的缺省参数值来自 Shape class 而非 Rectangle class!

为什么 C++ 坚持这种做法呢？答案在于运行期效率。如果缺省参数值是动态绑定，编译器就必须有某种办法在运行期为 virtual 函数决定适当的缺省参数值。这比目前所实行的“在编译器决定”的机制更慢且更复杂。



如果现在尝试遵守这条规则，并且同时提供缺省参数值给 base 和 derived classes ，又会发生什么？

```C%2B%2B
class Shape{
public:
    enum ShapeColor {Red, Green, Blue };
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};

class Rectangle: public Shape {
public:
    virtual void draw(ShapeColor color = Red) const;
    ...
};
```

可以看出，代码重复了，代码还带着相依性，如果 Shape 内的缺省参数值改变了，所有“重复给定缺省参数值”的那些 derived classes 也必须改变，否则它们又会导致“重复定义一个继承而来的缺省参数值”，那怎么办？

聪明的做法是考虑替代设计，条款 35 列了不少 virtual 函数的替代设计，其中之一是 NVI（non-virtual interface）手法：令 base class 内的一个 public non-virtual 函数调用 private virtual 函数，后者可被 derived classes 重新定义。这里可以让 non-virtual 函数指定缺省参数，而 private virtual 函数负责真正的工作：

```C%2B%2B
class Shape{
public:
    enum ShapeColor {Red, Green, Blue};
    void draw(ShapeColor color = Red) const       // 如今它是 non-virtual
    {
        doDraw(color);                            // 调用一个 virtual
    }
    ...
private:
    virtual void doDraw(ShapeColor color) const=0;  // 真正的工作在此完成
};

class Rectangle: public Shape {
public:
    ...
private:
    virtual void doDraw(ShapeColor color) const;      // 不需要指定缺省参数值
};
```

#### 37.4 总结

绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而 virtual 函数却是动态绑定。



### 条款 38：通过复合塑模出 has-a 或"根据某物实现出"

复合（composition）是类型之间的一种关系，当某种类型的对象内含它种类型的对象，便是这种关系。例如：

```C%2B%2B
class Address { ... };    // 某人的住址
class PhoneNumber { ... };

class Person {
public:
    ...
private:
    std::string name;    // 合成成分物
    Address address;
    PhoneNumber voiceNumber
    PhoneNumber faxNumber;
};
```

条款 32 说的是，“public 继承”带有 is-a（是一种）的意义。

复合也有它自己的意义，实际上它有两个意义：

-  has-a（有一个）
- is-implemented-in-terms-of（根据某物实现出）

当涉及到复合时，我们需要在软件中处理两个不同的领域：

1. 程序中的对象其实相当于我们所塑造的世界中的某些事物，例如人，汽车，一张张视频画面等等。这样的对象属于应用域（application domain）部分。
2. 其他对象则纯粹是实现细节上的人工制品，像是缓冲区（buffers）,互斥器（mutexes），查找树（search trees）等等。这些对象相当于我们软件的实现域（implementation domain）。

当复合发生于应用域内的对象之间，表现出 has-a 的关系；当它发生于实现域内则是表现 is-implemented-in-terms-of 的关系。

上述的 Person class 是一种 has-a 关系。Person 有一个名称，一个地址，以及语音和传真两笔电话号码。我们不会认为“人是一个名称”或“人是一个地址”，而会认为“人有一个名称”和“人有一个地址”。这就是 is-a 和 has-a 的关系。

比较麻烦的是区分 is-a（是一种）和 is-implemented-in-term-of（根据某物实现出）这两种对象关系。

#### 38.1 问题

假设我们需要一个 templates，希望制造出一组 classes 用来表示由不重复对象组成的 sets。

可能最容易想到的就是标准程序库提供的 set template。不幸的是 set 的每个元素都由“三个指针组成”，会导致额外的开销。因为 sets 通常以平衡查找树实现而来，使它们在查找，插入，删除元素时保证拥有对数时间效率。这个设计适用于速度比空间重要。

#### 38.2 错误做法

当我们对空间的要求大于对速度的要求时，就需要自己去写个 template。实现 sets 的方法太多，其中之一就是在底层采用 linked lists。而且标准库有一个 list template，于是我们决定复用它：

```C%2B%2B
template<typename T>        // 将 list 应用于 Set 。错误做法
class Set::public std::list<T> { ... };
```

条款 32 告诉我们，public 继承意味着 is-a （是一种）。意味着 D 是一种 B，对 B 为真的每一件事情对 D 也应该为真。对于上述例子，list 可以包含重复元素，而 Set 不可以包含重复元素。因此这两个 classes 之间并非 is-a 的关系，所以 public 继承不适合用来塑模它们。

#### 38.3 正确做法

正确做法是 Set 对象可根据一个 list 对象实现出来：

```C%2B%2B
template<class T>             // 将 list 应用于 Set，正确做法
class Set {
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;
private:
    std::list<T> rep;              // 用来表述 Set 的数据
};
```



#### 38.4 总结

复合（composition）的意义和 public 继承完全不同。

在应用域（application domain），复合意味 has-a（有一个）。在实现域（implementation domain），复合意味 is-implemented-in-terms-of（根据某物实现出）



### 条款 39：明智而审慎地使用 private 继承

private 继承意味着什么？观察下列代码：

```C%2B%2B
class Person { ... };
class Student:private Person { ... };
void eat(const Person& p);              // 任何人都会吃
void study(const Student& s);           // 只有学生才在校学习

Person p;              // p 是人
Studeng s;             // s 是学生

eat(p);                // 没问题，p 是人，会吃
eat(s);                // 错误，难道学生不是人？
```

显然 private 继承并不意味着 is-a 关系，那么它意味着什么？

如果 classes 之间的继承关系是 private：

1. 编译器不会将一个 derived class 对象（例如 Student）转换为一个 base class 对象（例如 Person）。这就是通过 s 调用 eat 会失败的原因。
2. 由 private base class 继承而来的所有成员，在 derived class 中都会变成 private 属性。

Private 继承意味着 implemented-in-terms-of（根据某物是现出）。如果让 class D 以 private 形式继承 class B，意思是采用 class B 内已经备妥的某些特性，不是因为 B 对象和 D 对象存在任何观念上的关系。

Private 继承纯粹只是一种实现技术，借用条款 34 提出的术语，private 继承意味着只有实现部分被继承，接口部分因略去。如果 D 以 private 形式继承 B，意思是 D 对象根据 B 对象实现而得，在没有其他意蕴了。Private 继承在软件“设计”层面没有意义，其意义只在软件实现层面。



Private 继承意味着 is-implemented-in-terms-of（根据某物实现出），复合（条款 38）的意义也是这样。那我们在两者之间如何取舍呢？答案是：尽可能使用复合，必要时才使用 private 继承，何时算必要呢？

1. 主要当 protected 成员 or virtual 函数被牵扯进来时。
2. 当空间方面的利害关系足以踢翻 private 继承。

#### 问题

现在我们有以下需求：我们有一个类 Widget class，我们需要记录每个成员函数的被调用次数，为完成这项工作，我们需要设定某种定时器，使我们知道收集统计数据的时候是否到了。如下所示：

```C%2B%2B
class Timer {
public:
    explicit Timer(int tickFrequency);
    virtual void onTick() const;    // 定时器每滴答一次
    ...                             // 此函数就被自动调用一次
                              
}
```

#### private 继承

为了让 Widget 重新定义 Timer 内的 virtual 函数，Widget 必须继承自 Timer。public 继承显然不合适，因为 Widget 并不是个 Timer。我们必须 private 继承：

```C%2B%2B
class Widget: private Timer{
private:
    virtual void onTick() const;  // 查看 Widget 的数据等等
};
```

#### 复合

要以复合取代 private 继承，只需在 Widget 内声明一个嵌套式 private class，后者以 public 形式继承 Timer 并重新定义 onTick，然后放一个这种类型的对象于 Widget 内。如下所示：

```C%2B%2B
class Widget {
private:
    class WidgetTimer: public Timer {
    public:
        virtual void onTick() const;
        ...
    };
    WidgetTimer timer;
};
```

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NTA4YzA3OWE2YzQ0OGZiZGJjOTczZTk2YWJlYzBhOGVfYnA0RzVqcVFwajdHRTJaMVlKaHN0OEFpeEtLWHV2VXdfVG9rZW46Ym94azRjM1NtWkViODBFZ2FVVVZjcGtreTdiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

提出本例子也是为了说明一个道理，训练自己思考多种做法是值的的。

选择复合而不是 private 的理由如下：

1. 这样可以让 Widget 拥有 derived classes，但同时可以阻止 derived classes 重新定义 onTick。（条款  35 曾说，derived classes 可以重新定义 virtual 函数）。但如果 WidgetTimer 是 Widget 内部的一个 private 成员并继承 Timer，Widget 的 derived class 将无法取用 WidgetTimer，因此无法继承它或重新定义它的 virtual 函数。这个方法可以用来模拟“阻止 derived classes 重新定义 virtual 函数”。
2. 将 Widget 的编译依存性降至最低。如果 Widget 继承 Timer，当 Widget 被编译时 Timer 的定义必须可见。但如果 WidgetTimer 移出 Widget 之外而 Widget 内含指针指向一个 WidgetTimer，Widget 可以只带着一个简单的 WidgetTimer 声明式。



为什么使用 private 继承：

1. 一个类想成为一个 derived class 且想访问一个 base class 的 protected 成分，或为了重新定义一或多个 virtual 函数。这时候两个 classes 之间的概念关系其实是 is-implemented-in-terms-of（根据某物实现出）而非 is-a。
2. 另一种情况较为激进，只适用于所处理的 class 不带任何数据。这样的 class 没有 non-static 成员变量，没有 virtual 函数，也没有 virtual base classes。于是这种所谓的 empty classes 对象不使用任何空间，因为没有任何隶属于对象的数据需要存储。然后由于技术上的理由，C++ 里凡是独立对象都必须有非零大小。

```C%2B%2B
class Empty {};     // 没有数据

class HoldsAnInt {
private:
    int x;
    Empty e;
};
    
```

sizeof(HoldsAnInt) > sizeof(int)，在大多数编译器中 sizeof(Empty)=1，对于这样的“大小为零的独立对象”，C++ 官方默默安插一个 char 到空对象内。

上述所谓的独立，其对立面非独立指的是 derived class 对象内的 base class 成分：

```C%2B%2B
class HoldsAnInt:private Empty {
private:
    int x;
};
```

几乎可以确定 

sizeof(HoldsAnInt) > sizeof(int)。这就是所谓的 EBO（empty base optimization：空白基类最优化）：这个性质对于程序库开发人员而言可能有用，由于其客户特别在意空间。EBO 一般只在单一继承下可行。



现实中的 "empty" classes，并不真的是 empty，虽然它们没有 non-static 成员变量，但却往往含有 typedefs，enums，static 成员变量，或 non-virtual 函数。

STL 许多技术用到 empty classses，其中内含有用的成员（通常是 typedefs），包括 base classes unary_function 和 binary_function，这些是“用户自定义函数对象”通常会继承的 classes。



#### 总结

Private 继承意味着 is-implemented-in-terms of（根据某物实现出）。它通常比复合（composition）的级别低。当 derived classes 需要访问 protected base class 的成员，或者需要重新定义继承而来的 virtual 函数时



和复合不同，private 继承可以造成 empty base 最优化。



### 条款 40：明智而审慎地使用多重继承

多重继承指继承一个以上的 base classes，这些 base classes 又有更高级的 base classes。某种情况下可能导致钻石继承问题。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=OWMyYWMyMGIzNzM4ZWI2ODAxYmQzOGY1ODRlOGI4NGNfbUJCcjhsWTloeDJEc2Q5dU1rbzBRVUdsR0RrV0ZRdENfVG9rZW46Ym94azQyaXFoV2FOMlhTUElsWHR2UDdJb2pnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

任何时候，如果某一个继承体系中某个 base class 和某个 derived class 之间有一条以上的相通路线，就必须面对以下问题：

1. 是否让 base class 内的成员经由每一条路被复制，假设 File class 有个成员变量 fileName，这样 IOFile 将有两份 fileName。
2. 从现实角度看， IOFile 对象只该有一个文件名称，所以它继承自两个 base classes 而来的 fileName 不该重复。

C++ 对于这两个方案都支持，但其默认做法是方案1。如果想要方案2，就必须让带有此数据的 class（File）成为一个 virtual base class。为了这样做，必须令所有直接继承自它的 classes 采用“virtual 继承”：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MzZlNWU1M2U5NDAyNmIxZTM4YzY4MjA4Mjg4NzJlYmZfdWxlNzEyUjAzYW5vS0RpWkNsOFo5SzJHNU16cGlka1pfVG9rZW46Ym94azRQQnlEZ1R2RkpzSlRha0hMUzhhakFjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

注意：使用 virutal 继承的那些 classes 所产生的对象往往比使用 non-virtual 继承的对象体积更大，访问 virtual base classes 的成员变量时，也比访问 non-virtual base classes 的成员变量速度慢。



支配 "virtual base classes 初始化"的规则比起 non-virtual bases 的情况更复杂且不直观。virtual base 的初始化责任是由继承体系中的最低层 class 负责。这表明：

- classes 若派生自 virtual bases 而需要初始化，必须明确其 virtual bases——不论那些 bases 距离多远
- 当一个新的 derived class 加入继承体系，它必须承担其 virtual bases 的初始化责任。



建议：

1. 非必要不使用 virtual bases
2. 如果必须使用 virtual base classes，尽可能避免在其中放置数据。这样就不涉及到这些 virtual bases 的初始化和赋值等行为。



以下是一个使用多重继承的例子：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MTcwYjViM2NhNmMxMDEzNjg1MjI2MDAzZmYzZmE5MTRfY2JsUUhZVEdaREZod00xTE4xMkRaaHpLUmJTWTAwZWlfVG9rZW46Ym94azRxTVl6bjVvNjNpT1FJQzBobHRGaVBoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YzJjNTA4M2U2ZGZjM2M1ZGUwYWE3MjhiZDI0OWM3NzlfQlZxbXprbk9ZQkVsT2lyaUthNk5heU91b0JoTUxkeVJfVG9rZW46Ym94azRlUXJLRWRzNnNkMmxPRm5ocEh5ZW1FXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

在 UML 图中这个设计看起来是这样：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MGNkN2M0MzA3NWJmOGZiYjBlMDMxZDEwMDkxMTllODFfdXQ5cW5TZzdxUGRGT0tEV2hnajJtQlZvdG56Y3B2ak5fVG9rZW46Ym94azRtT2JpWVUzcWY2NnJ6eEc0ODJhaDJnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述代码中的多重继承将“public 继承自某接口”和“private 继承自某实现”结合起来。



#### 总结

- 多重继承比单一继承复杂。为了避免歧义，应该使用 virutal 继承。
- virtual 继承会增加大小，速度，初始化复杂度等等成本。如果 virtual base classes 不带任何数据，将是最具实用价值的情况。
- 多重继承的典型用途如：涉及“public 继承某个 Interface class”和 "private 继承某个协助实现的 class"的两相组合。



## 七、模板与泛型编程

C++ templates 最初发展的动机是：建立“类型安全”的容器，如 vector，list 和 map。容器当然很好，但基于 templates 的泛型编程——写出的代码和其所处理的对象类型彼此独立则更好。

最终，人们发现，C++ template 机制自身是一部完整的图灵机：它可以被用来计算任何可计算的值。于是产生了模板元编程（template metaprogramming），创作出“在 C++ 编译器内执行并于编译完成时停止执行”的程序。

容器只是 C++ template 应用的一小部分，有一组核心观念一直支撑着所有基于 template 的编程，这些观念是本章讨论的焦点。



### 条款 41：了解隐式接口和编译期多态



面向对象编程世界总是以显式接口（explicit interfaces）和运行期多态（runtime polymorphism）解决问题。

#### 41.1 运行期多态与显式接口

```C%2B%2B
class Widget {
public:
    Widget();
    virtual ~Widget();
    virtual std::size_t size() const;
    virtual void normalize();
    void swap(Widget& other);
    ...
};

void doProcessing(Widget& w){
    if(w.size() > 10 && w != someNastyWidget){
        Widget temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```

对于函数 doProcessing：

- w 的类型为 Widget，所以 w 必须支持 Widget 接口，我们可以在源代码中找出这个接口，看看它是什么样子，这样的接口称为显式接口。
- 由与 Widget 的某些成员函数是 virtual，w 对那些函数的调用将表现出运行期多态，也就是说将于运行期根据 w 的动态类型（见条款 37）决定调用哪一个函数。



#### 41.2 隐式接口与编译期多态

Template 及泛型编程的世界，与面向对象有根本上的不同。在此世界中显式接口和运行期多态仍然存在，但重要性降低，关键在于隐式接口（implicit interfaces）和编译期多态（compile-time polymorphism）。为了解释它们，我们现在将  doProcessing 从函数转变成函数模板（function template）：

```C%2B%2B
template<typename T>
void doProcessing(T& w){
    if(w.size() > 10 && w != someNastyWidget){
        Widget temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```

对于 doProcessing，我们可以说：

- w 必须支持哪一种接口，是由 template 中执行于 w 身上的操作来决定。例如，本例中 w 的类型 T 好像必须支持 size, normalize 和 swap 成员函数，copy 构造函数，不等比较。其实这并不完全准确，重要的是，这一组表达式（对此 template 而言必须有效编译）是 T 必须支持的一组隐式接口。
- 模板的具现化发生于编译期，“以不同的 template 参数具现化 function templates”会导致调用不同的函数，这便是编译期多态。

显式接口由函数的签名式（也就是函数名称，参数类型，返回类型）构成。

隐式接口并不基于函数签名式，而是有效表达式（valid expressions）组成，再次看 doProcessing template 一开始的条件：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZjNmNTQ2NmIwOTBjNWIxYWZiOWIwODk5MDY0NzQ3NzFfd0tLeGZLb0pCWHNjSHQxdXdxZTNYRmlqWmNvalV3dWhfVG9rZW46Ym94azRQd0ExcFQzbVdFZm9IV05nWTRVQk5jXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

T（w 的类型）的隐式接口看来好像有这些约束：

- 它必须提供一个名为 size 的成员函数，该函数返回一个整数值。
- 它必须支持一个 operator!= 函数，用来比较两个 T 对象。

因为操作符重载的存在，这两个约束都不需要满足。T 必须支持 size 成员函数，这个成员函数不需要返回一个整数值，甚至不需要返回一个数值类型，它甚至不需要返回一个定义有 operator> 的类型，它唯一需要做的是返回一个类型为 x 的对象，而 x 对象加上一个 int（10的类型）必须能够调用一个 operator>。这个 operator> 不需要非得取得一个类型为 x 的参数不可，它也可以取得类型为 Y 的参数，只要存在一个隐式转换能构将类型为 X 的对象转换为类型为 Y 的对象。

同样，T 并不需要支持 operator!=，如果 operator!= 接受一个类型为 X 的对象和一个类型为 Y 的对象，T 可被转换为 X 而 someNastyWidget 的类型可悲转换为 Y，这样就可以调用 operator!=

隐式接口仅仅是由一组有效表达式构成，表达式自身可能看起来很复杂，但它们要求的约束条件一般而言相当直接又明确。例如以下条件式：

if(w.size() > 10 && w != someNastyWidget)

从整体上看，if 语句的条件式必须是个布尔表达式，无论 “w.size() > 10 && w != someNastyWidget”导致什么，它都必须与 bool 兼容。

这是 template doProcessing 中，类型参数 T 的隐式接口的一部分。doProcessing 要求的其他隐式接口：copy 构造函数，normalize 和 swap 也都必须对 T 型对象有效。



template 参数身上的隐式接口和 class 对象身上的显式接口都在编译期完成检查，无法在 template 中使用“不支持 template 要求的隐式接口”的对象（代码通不过编译）



#### 41.3 总结

- classes 和 templates 都支持接口和多态
- 对 classes 而言接口是显式的，以函数签名为中心。多态则是通过 virtual 函数发生于运行期
- 对 template 参数而言，接口是隐式的，奠基于有效表达式。多态则是通过 template 具现化和函数重载解析（function overloading resolution）发生于编译期。



### 条款 42：了解 typename 的双重意义

#### 42.1 typename 与 class

提一个问题，以下 template 声明式中，class 和 typename 有什么不同？

```
template<class T> class Widget;
template<typename T> class Widget; 
```

答案：没有不同。当我们声明 template 类型参数，class 和 typename 的意义完全相同。作者比较喜欢 typename，因为它暗示参数并非一定得是个 class 类型。



#### 42.2 typename 作为嵌套从属类型名称的前缀词

C++ 并不总是把 class 和 typename 视为等价。有时候你一定得使用 typeame。现在谈谈在 template 中涉及到的两种名称：

假设先有一个 template function，接受一个 STL 兼容容器为参数，进一步假设这个函数只是打印其第二个元素值，注意该模板函数无法通过编译，稍后解释原因：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NzU1Njg4OGE4YzMxYWY2YzU5MDdjNzhjZTQ5ZTZiODFfdXB1OHJma0QxY0V1Y0EyeGpCOXRUbGhURE16NDlPZThfVG9rZW46Ym94azRyZU1CSGpIcWsyYUpjR0x2RllscTliXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

代码中有两个 local 变量 iter 和 value。iter 的类型是 C::const_iterator，实际是什么取决于 template 参数 C。

- template 内出现的名称如果相依于某个 template 参数，称之为从属名称（dependent names）。
- 如果从属名称在 class 内呈嵌套状，我们称它为嵌套从属名称（nested dependent name）。
- C::const_iterator 就是这样一个名称。实际上它还是个嵌套从属类型名称（nested dependent type name）,也就是个嵌套从属名称并且指涉某类型。
- 另一个 local 变量 value，其类型是 int。int 是一个并不倚赖任何 template 参数的名称，这样的名称是非从属名称（non-dependent names）。



嵌套从属名称可能导致解析（parsing）困难，假设我们令上述模板函数更愚蠢些：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MTgxYzczMDg4ZDk0NTE4Y2UyYjFlMzEzZTc5YWRjYTNfS2ZnQmtzVzdGQzd6TWFMMGJQYUltdGZ4N3RDcTlCZ05fVG9rZW46Ym94azQ5clI4M0lyd3FoWXlqc01PRXFXQ1plXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

看起来好像声明 x 为一个 local 变量，它是一个指针，指向一个 C::const_iterator。我们之所以这么认为，只因为我们“已经知道”C::const_iterator 是个类型。如果 C::const_iterator 不是个类型呢？如果 C 有个 static 成员变量而碰巧被命名为 const_iterator，如果 x 碰巧是个 global 变量名称呢？那样的话上述代码就不再是声明一个 local 变量，而是一个相乘动作。

C++ 编译器的作者必须操心所有可能的输入，甚至是上面这么疯狂的输入。在我们知道 C 是什么之前，无法知道 C::const_iterator 是否为类型，C++ 有个规则可以解析此为歧义转态：如果解析器在 template 中遭遇一个嵌套从属名称，它便假设这名称不是类型，除非明确告诉它是。



现在再看 print2nd：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MTgxYzczMDg4ZDk0NTE4Y2UyYjFlMzEzZTc5YWRjYTNfS2ZnQmtzVzdGQzd6TWFMMGJQYUltdGZ4N3RDcTlCZ05fVG9rZW46Ym94azQ5clI4M0lyd3FoWXlqc01PRXFXQ1plXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

现在应该很清楚为什么这不是有效的 C++ 代码了。 iter 声明式只有在 C::const_iterator 是个类型时才合理，但我们并没有告诉 C++ 它是。解决办法是：使用 typename 告诉C++ 编译器 C::const_iterator 是个类型。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NjFmNmQyNGQ0MTkwZTEyNmMzMjQ4ZTM3NDczOTIyNGFfeEF0NnlQbHV2SXE3azFSUmd4V3AzanMyaUpvZjRvM2NfVG9rZW46Ym94azRCcGRYajRqelYzTjA5VEFhNk16TlNjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



一般规则很简单，任何时候想要在 template 中使用一个嵌套从属类型名称，就必须在紧临它的前面放上关键字 typename。

typename 只被用来证明嵌套从属类型名称：前它名称不该有它存在。例如：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZjhjYTVhNDMwNWJhN2EyOThhYTdlYjI4MGFjMGVhNGFfRXA2dUY1U0RaVTRDWHBMQ1JLNjdobG83WEx6VDBaQ3dfVG9rZW46Ym94azRpN2k3R0VxYkhNeG1GZ2VhejVueWlmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述的 C 并不是嵌套从属类型名称（并非嵌套于任何“取决于 template 参数”的东西内），所以声明 container 时并不需要以 typename 为前导，但 C::iterator 是个嵌套从属类型名称，所以必须以 typename 为前导。



#### 42.3 typename 作为嵌套从属类型名称的前缀词的例外

"typename 必须作为嵌套从属类型名称的前缀词"这一规则的例外是，typename 不可以出现在base classes list 内的嵌套从属名称之前，也不可以在 member initialization list（成员初始化列表）中作为 base class 修饰符，例如：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MzViM2I3ZmZhZjFiMjBjNjNmZTI1N2MzNWNhODFmYThfeDN6akJWcXpobHRPN1VDTklPenpnT0FnTUtvdVd6RWNfVG9rZW46Ym94azRLeUV1VHB6N3k1eGRtcmE5WWN0RkZmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



#### 42.4 代表性例子

该例子可能会在真实程序中碰到，假设我们正在编写一个 function template，它接受一个迭代器，而我们打算为该迭代器指向的对象做一份复件 temp，我们可以这么写：

```C%2B%2B
template<typename IterT>
void workWithIterator(IterT iter){
    typename std::iterator_traits<IterT>::value_type temp(*iter);
}
```

上述代码是 traits class （见条款 47）的一种运用，相当于“类型为 IterT 的对象所指之物的类型”。这个语句声明一个 local 变量 temp，使用 IterT 对象所指物的相同类型，并将 temp 初始化为 iter 所指物。如果 IterT 是 list<int>::iterator，temp 的类型就是 int，如果 IterT 是 vector<string>::iterator，temp 的类型就是 string。

value_type 被嵌套于 iterator_traits<IterT> 之内而 IterT 是个 template 参数，所以必须在它之前放置 typename。

如果“std::iterator_traits<IterT>::value_type”读起来不畅快，有经验的程序员应该会想到 typedef。对于 trait 成员名称如 value_type，使用 typedef 名称以代表某个 traits 成员名称，常常可以在实际的代码中看到这样的 local typedef：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YWQ5ZTg1MWI1MGQ1NjhkYjhkZmYwMjFjYWM0ODc5YzhfR0lNSVFZcmx6bGxVZ1dEUFpwd1gxS0ZLcURURHpGcERfVG9rZW46Ym94azRVcHltbm1uTVVJZkhZZkc1RnVVd3JiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



#### 42.5 总结

- 声明 template 参数时，前缀关键字 class 和 typename 可互换。
- 请使用关键字 typename 标识嵌套从属类型名称，但不得在 base class lists（基类列）或 member initialization list（成员初始化列表）内以它作为 base class 修饰符。



### 条款 43：学习处理模板化基类内的名称

假设需要编写以下程序，它能够传送信息到若干不同的公司去，可以发送加密信息，也可以发送明文信息。如果编译期间有足够信息来决定哪一个信息传至哪一家公司，就可以采用 template：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=OTAwNWJhMWM3OTFiOTQ0YmVkNGZkNWNkMzZlNWVlNDBfOFI5b1ozcTVYSnByVFU0ZmVKa1pTdHF1dEdaZ3NvWWtfVG9rZW46Ym94azRaY2JsYnROSldJRlM4SHU3QUFoRlBoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



这个做法行得通，假设我们需要在每次发送信息前后，标记（log）某些信息，derived class 可以轻易实现：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YTlmMzE1MWQyYmU5ZThmNzA4ZjZmNTUwZmE2MWE0ZjlfQzZUUUZEN3A4bHUyaU9xMWdYSE9HYUVqZHpVMVd6dmhfVG9rZW46Ym94azR6QkZpbXJhTTVCQ05PT0Z4YzY5TWNkXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

注意这个 derived class 的信息传送函数有一个不同的名称（sendClearMsg），与其base class 内的名称（sendClear）不同。这是个好设计，因为避免“名称遮掩”和“重新定义一个继承而来的 non-virtual 函数”。但上述代码无法运行，因为编译器不知道 sendClear 的存在。我们可以看到 sendClear 存在于 base class 内，编译器却看不到它们，为什么？

问题在于，当编译器遭遇 class template LoggingMsgSender 定义式时，并不知道它继承什么样的 class。当然是继承的是 MsgSender<Company>，但其中 Company 是个 template 参数。除非 LoggingMsgSender 被具现化，否则无法确切知道它是什么，因而也就无法知道它是否有 sendClear 函数。



#### 模板特化

为了让问题更具体化，假设现有个 class companyZ 坚持使用加密通讯：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YTYzNGRkN2Y2MDE3YjJmZmNlZWI1MTk3ZmRiMjViMWVfQ0lIc1hvZnJqMVRGc29DMXhhM3FYNDZFWGhRcEpyZk9fVG9rZW46Ym94azRrZlM4eWVhR013OHJrRnBLcEhocHFiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

很明显上述 CompanyZ 对一般性的 MsgSender template 并不合适，因为其并未定义 SendCleartext 函数。为了矫正这个问题，我们可以针对 CompanyZ 产生一个 MsgSender 特化版：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YWE2ODc3NmNmNjBlYWEyOTI5NGY0OTVmMzcxMThjOGZfQjFXblNReDZBNmpWV0o0TXQ5cWdCM0czM1BRb0twTkdfVG9rZW46Ym94azRpc2p3T01WajJjM1NBMldYa2pqOUJoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述 “template<>” 表示这既不是 template，也不是标准 class，而是个特化版的 MsgSender template，在 template 实参是 CompanyZ 时被使用。



现在，MsgSender 针对 CompanyZ 进行了全特化，现在再次考虑 derived class LoggingMsgSender：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YzQ0NWU4MWEwMDU4NDM2Y2E5MWIyZmRjMTVjZWY3ZDJfOWd5RWlZdmRUWmtrSGhXYmNXcmNUWXdUTnBraG1xYk1fVG9rZW46Ym94azRFZnByUlBCWWhaUTc3MDZVV3lTVlpEXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

当 base class 被指定为 MsgSender<CompanyZ> 时这段代码不合法，因为那个 class 并未提供 sendClear 函数。

这就是为什么 C++ 拒绝这个调用的原因：它知道 base class templates 有可能被特化，而那个特化版本可能不提供和一般性 template 相同的接口。因此它往往拒绝在 templatized base classes（模板化基类）内寻找继承而来的名称。

当我们从 Object oriented C++ 跨进 Template C++，继承会受到某些限制。

#### 解决办法

为了解决上述问题，我们必须有某种办法令 C++ 进入 templatized base classes，有三个办法。

1. 在 base class 函数调用的动作前加上 this->

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YmFhY2UyZDU1YzEwZjM3NDU5ZjM2NTM2MjQyMmExMmJfRE1JVW9TbkRqaHlKM2pqeVdDanp2VGFiZ1JyYkl1OEhfVG9rZW46Ym94azRaV0tCSW1OdEp4OGthcnVLQ1JJaE5lXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



1. 使用  using 声明式

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MzczMDI1ODdkZTVkNTRkNGUyNGRiYzgwNzA0M2NlYzZfY214cEUxSVBuV2pwdFUyZU1EVVA2eGhjZHNPU1hVUWtfVG9rZW46Ym94azREcW81ejJWM2NTVXVmN1B0dmpmT2pNXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这里表示编译器不进入 base class 作用域内查找，于是我们通过 using 告诉它，请它那么做。



1. 明白指出被调用的函数位于 base class 内：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZjRjNjE5NmE2MzYwYjAyY2U2MDNmN2VhYTNlNGY5ZmVfVWp2alpmbVdWcHFHR2ZuRk00Q1FlMXlDTkRib05IMEhfVG9rZW46Ym94azRVNkVjQjBaNUI5MVR1UERTZ3RQWmVlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

但这往往是最不让人满意的一个解法，因为如果被调用的是 virtual 函数，上述的明确资格修饰（explicit qualification）会关闭“virtual 绑定行为”。



如果在稍后的源码内包含以下代码：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NWQwYWQzZDNmZGI1ZmUzNGNjZDU1YWEzMjgwYmU3NmJfV3JBZHdVZWZFYUFHZ21nTjh4Q0Q1bVc0cDE5ZUpoM3ZfVG9rZW46Ym94azRRRHVUVlpmQ0wyOUExOFdoVk50VnVjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

其中对 sendClearMsg 的调用动作将无法通过编译，因为在那个点上，编译器知道 base class 是个  template 特化版本 MsgSender<CompanyZ>，而且知道那个 class 不提供 sendClear 函数。



编译器寻找代码错误的时间可能发生在早期（解析 derived class template 的定义式时），也可能发生在晚期（当那些 template 被特定的 template 实参具现化时）。C++ 的政策是宁愿较早诊断。



#### 总结

可在 derived class template 内通过 "this->" 完成对 base class templates 内的成员的访问，或由一个写出的“base class 资格修饰符” 完成。



### 条款 44：将与参数无关的代码抽离 templates

Template 是节省时间和避免代码重复的一个好方法，class templates 的成员函数只有在被使用时才暗中具现化。但必须注意，使用 templates 也可能导致代码膨胀（code bloat）：其二进制码带着重复（或几乎重复）的代码，数据。



为了避免这个问题，我们通常使用一个叫做：共性与变形分析（commonality and variability  analysis）的方法。其概念也很简单，看下列例子：

- 当我们编写某个函数，其中的实现码和另一个函数的实现码实质相同，我们会抽出两个函数的共同部分，把它们放进第三个函数中，然后令原先两个函数调用这个新函数。（也就是找出共同的部分和不变的部分）
- 当我们编写某个 class，其中的一部分和另一个 class 的某些部分相同，我们会把共同的部分搬到新 class 中去，然后使用继承或复合（见条款 32，38，39）令原先的 classes 取用这些共同特性。



编写 templates 时，也做相同的分析，以相同的方式避免重复，有以下窍门：

在 template 代码中，重复是隐晦的：毕竟只存在一份 template 源码，所以必须训练自己去感受当 template 被具现化多次时可能发生的重复。

#### 44.1 问题

举个例子：假如我们想为固定尺寸的正方形矩阵编写一个 template，该矩阵的性质之一是支持逆矩阵运行（matrix inversion）。

#### 44.2 方案一

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZTVlNzdjMzBmYzM0YmU2NTE2YmM5YjhmZDVkZGNiOTJfdHRyRGtyQ3dKZVJ4NHJDSDlXNGtZMnE5enRqckFHdzhfVG9rZW46Ym94azRWejVDdTdSSXF0YTlNM1BwbmlGVFN2XzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这个 template 接受一个类型参数 T，除此之外还接受一个类型为 size_t 的参数，这是个非类型参数（non-type parameter），这种参数和类型参数比起来较不常见，但它们完全合法。

现在，观察这些代码：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NGExYjA1NTE0M2VjMTgyNjU5ODEzOTcwMWQyNDUyNDRfbXRidGdoa1d6OGJPNHJIcmZaZHpMSGV3RFozT05WYUJfVG9rZW46Ym94azRXajAyd1JTN0ZNWkt3Zk1JbVJHU2dlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述代码会具现化两份 invert，但这两份 invert，除了常量 5 和 10，两个函数的其他部分完全相同。这是 template 引发代码膨胀的一个典型例子。



#### 44.3 方案二

##### 关于 invert

第一次修改，我们将创建一个带数值参数的函数，然后以 5 和 10 来调用这个带参数的函数，而不重复代码：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YjY2ODczZGE0MGQxNTFjZWIzZjNlMGIyOWNmMTRkYjFfd3dIYlEyVkdGWm9oUVdBSHl6VDlaYXR3cGE0TkNHWUJfVG9rZW46Ym94azRPaGNFc3hCUE1GT0tOT1NCc0Ryd2lnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

带参数的 invert 位于 base class SquareMatrixBase 中，和 SquareMatrix 一样，base class SquareMatrixBase 也是 template，不同的是它只对"矩阵元素的类型"参数化，不对矩阵的尺寸参数化。因此，对于元素对象类型相同的矩阵，它们都共享同一个 SquareMatrixBase ，它们也将共享这唯一一个 class 内的 invert。



- SquareMatrixBase::invert 只是企图“避免 derived classes 代码重复”的一种方法，所以它是 protected。
- 调用 SquareMatrixBase::invert 成本应该是 0，因为 derived classes 的 invert 调用 base class 版本时用的 inline

关于 "this->"，原文说的是如果不使用，模板化基类内的函数名称会被 derived classes 遮掩。



注：其实，在这里不是用 "this->" 也可以，因为前面已经使用了 using 让 base class 内的 invert 在该类中可见，而且 SquareMatrixBase::invert 和 SquareMatrix::invert，存在重载关系，系统会选择合适的函数执行。



##### 矩阵存储在哪

这一次我们要面对的问题是 SquareMatrixBase::invert 如何知道该操作什么数据？它如何知道哪个特定矩阵的数据在哪儿？

一个可能的做法是为 SquareMatrixBase::invert 添加另一个参数，指向一块用来放置矩形数据的内存起点。如果有若干这样的函数，我们可以对所有这样的函数添加一个额外参数，却得一次又一次地告诉 SquareMatrixBase 相同的信息，这似乎不好。



另一个办法是令 SquareMatrixBase 存储一个指针，指向矩阵数值所在的内存。顺便也可以在  SquareMatrixBase 存储矩阵的尺寸。如下所示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZTg5MzYxZTg0NGYxNTZiODMxYTRhM2M0ZTk2ZTkxMmNfNnVjSTNXRTExOTNwVU05ejN0a0xNcHE0WnVKVFZiQTZfVG9rZW46Ym94azRvaFpyaFZoSW43ZjB5ejFwbmpyT0hlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这允许 derived class 决定内存分配方式。某些实现版本也许会将矩阵数据存储在 SquareMatrix 对象内部：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NzM3OWRjOWJlM2ZjOTYwZGYwM2U1MDI5YWRiNjc5ZTRfcVFUVGxKdHo3M0hlUlVoNDNRMzhrbllCRk05ZFJmdlFfVG9rZW46Ym94azRrR3B0OGw5WWlwVkQ5S213NW5pUnRlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这种类型的对象不需要动态分配内存，但对象自身可能非常大。另一种做法是让每一个矩阵的数据放进 heap：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=M2EyZmEwZDc0ZmUzMzY1M2I3MGVlMjhkNjI2YWVlNTNfdGx1aTFNNjZvSG82dVR1ZDRHSTBWZkJFcXkzQ0NCOVpfVG9rZW46Ym94azQzak9sNkhTWVQ3RUFVQXp3VjhCeUtiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

#### 取舍

在上面所述的方案中，不同大小的 SquareMatrix 对象有着不同的类型，所以即使（例如：SquareMatrix<double,5> 和 SquareMatrix<double, 10>）对象使用相同的 SquareMatrixBase<double> 成员函数，我们也没机会传递一个 SquareMatrix<double,5> 对象到一个期望获得 SquareMatrix<double, 10> 的函数去。

##### 方案一优点

如前所述，方案一中的 invert 版本，有可能生成比共享版本（其中尺寸以函数参数传递或存储在对象内）更佳的代码。例如在尺寸专属版（方案一）中，尺寸是个编译期常量，因此可以由常量的广传达到最优化，包括把它们折进被生成指令中成为直接操作数。这一特性是，“与尺寸无关”版本（方案二）无法办到的。



##### 方案二优点

从另一个角度，不同大小的矩阵只拥有单一版本的 invert，可减少执行文件大小，也就因此降低程序的 working set 大小，并强化指令高速缓存区内的引用集中化（locality of reference）。这些都可能使程序执行得更快速，超越“尺寸专属版”invert 的最优化效果。

> 所谓 working set 是指在一个“虚内存环境”下执行的进程而言，其所使用的那一组内存页（pages）



##### 决择

那到底该选择哪一个版本更好呢？唯一的办法是两者都尝试并观察我们所使用平台的行为以及面对代表性数据组时的行为。

##### 

#### 总结

本条款只讨论由 non-type template parameters（非类型模板参数）带来的膨胀，其实 type parameters（类型参数）也会导致膨胀。例如：

- 在许多平台，int 和 long 有相同的 二进制表示，所以像 vector<int> 和 vector<long> 的成员函数有可能完全相同。有些连接器（linkers）会合并完全相同的函数实现，但有些不会。后者意味着 template 被具化为 int 和 long 两个版本，并因此可能造成代码膨胀。
- 类似的，在大多数平台，所有指针类型都有相同的二进制表述，因此凡 template 持有指针者，（例如 list<int*>, list<const int*>, list<SquareMatrix<long, 3>*>等等）往往应该对每一个成员函数使用唯一一份底层实现。

如果我们实现某些函数而它们操作强型指针（srongly typed pointers，即 T*），应该令它们调用另一个操作无类型指针（untyped pointers，即 void*）的函数，由后者完成实际工作。



- Templates 生成多个 classes 和多个函数，所以任何 template 代码都不该与某个造成膨胀的 template 参数产生相依关系。
- 因非类型参数（non-type template parameters）而造成的代码膨胀，往往可以消除，做法是以函数参数或 class 成员变量替换 template 参数。
- 因类型参数（type parameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述（binary representations）的具现类型（instantiation types）共享实现码。



### 条款 45：运用成员函数模板接受所有兼容类型

真实指针做得很好的一件事是：支持隐式类型转换（implicit conversions）。Derived class 指针可以隐式转换为 base class 指针，“指向 non-const 对象”的指针可以转换为“指向 const 对象”。如下所示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ODYzODQ2MGE5ZGM5MGQyMThiYzhjNTk2Yzg3NmVhYjlfWGViV1hEUnFMOEZ5UFpvM2NmUGJQU0RXR1NhZHlodkNfVG9rZW46Ym94azQzSVllUW1ieUVWeHo1NldPbXVKTWNoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

如果用户想在自定义的智能指针中模拟上述转换，稍微有点麻烦，我们希望如下代码通过编译：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NDBmNzQwN2ZjY2JjMzRlZjEzNmNiNDMwYzhkNmUxY2NfZWw1dHlqSWhydVFwc1Q1WnpIOU43eVEwbEZSRER4RWdfVG9rZW46Ym94azROQnJMY25qTWhjR3F6RzFuVnNiZjdkXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

同一个 template 的不同具现体（instantiations）之间并不存在什么与生俱来的固有关系，所以编译器视 SmartPtr<Middle> 和 SmartPtr<Top> 为完全不同的 classes。

为了获得我们希望的 SmartPtr classes 之间的转换能力，我们必须将它们明确地编写出来。



#### Templates 和 泛型编程（Generic Programming）

从上述代码示例中可以看出，每个语句都创建了一个新式智能指针对象，所以现在我们重点关注如何编写智能指针的构造函数，使其行为满足我们的转换需要。如下所示：

```C%2B%2B
template<typename T>
class SmartPtr {
public:
    SmartPtr(const SmartPtr<Middle>& other);
    SmartPtr(const SmartPtr<Bottom>& other);
    ...
};
```

上述代码我们可以根据一个 SmartPtr<Middle> 或一个 SmartPtr<Bottom> 构造出一个 SmartPtr<Top>。假设这个继承体系未来所有扩充，SmartPtr<Top> 对象又必须能够根据其它指针指针构造自己，例如日后添加了：

class BelowBottom: public Bottom { ... }

我们又必须写出以下构造函数：

SmartPtr(const SmartPtr<BelowBottom>& other);

一个很关键的观察结果是：我们永远无法写出我们需要的所有构造函数。



现在我们尝试写一个 Template 来满足所有类似需求，因为一个 template 可被无限量具现化，以至生成无限量函数。因此我们需要为它写一个构造模板，这样的模板（templates）是所谓的 membe function template（常简称 member templates），其作用是为 class 生成函数：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YWJmN2RlMTA1MjEyZTAzNzlkODdhZjliOGE2N2RmOWVfTWl2MDZFMWtjSjNWWGRzMmhFT1NMaUo2b0FBcEZoN1ZfVG9rZW46Ym94azRiUFNtZHE4R1pWblR2aW9EZXE3alJjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

以上代码意思是，对任何类型 T 和任何类型 U，可以根据 SmartPtr<U> 生成一个 SmartPtr<T>。因为它的构造函数模板产生的构造函数可以根据对象  u 创建对象 t（例如根据 SmartPtr<U> 创建一个 SmartPtr<T>），而 u 和 t 的类型是同一个 template 的不同具现体，有时我们称之为泛化（generalized）copy 构造函数。



上面的泛化 copy 构造函数并未被声明为 explicit，那是故意的，因为原始指针类型之间的转换是隐式的，所以让智能指针效仿这种行为是合理的。



由于上述为 SmartPtr 所写的“泛化 copy 构造函数”提供的东西比我们需要的更多。我们不希望根据一个 SmartPtr<Top> 创建一个 SmartPtr<Bottom>，因为那对 public 继承而言是矛盾的。

假设我们提供一个 get 成员函数，返回智能指针对象所持有的那个原始指针的副本，那么我们可以在“构造模板”实现代码中约束转换行为。如下：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NjgyYWMxYmQ4ZmY0Y2VlOTJhNzYxMTk3ODNjMGRhYjlfYmd5aEhYMnFwYW45TWhVRVFmR1FUTXVkc21zRG5xbGpfVG9rZW46Ym94azRRNzc1NXk1RGxjUUVUSEZMS0c3UjB5XzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

我们使用成员初始化列表（member initialization list）来初始化 SmartPtr<T> 中的 T* 成员变量，并以类型为 U* 的指针（由 SmartPtr<U>持有）作为初值。这个行为只有当“存在某个隐式转换可将一个 U* 指针转为一个 T* 指针”时才能通过编译，而那正是我们想要的。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=OGU1ODJhZjkxZGYyNDYzMjY5OGY2NjViZTY2YjRiOTFfRnhnRm9KcEdWbld5dkVET2JpRlNaUVJ5bTlqUnIzUU1fVG9rZW46Ym94azRXaTBqRGQ0andDME9QTDgxY01yNEllXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

现在 SmartPtr<T> 有了一个 泛化 copy 构造函数，这个构造函数只在其所获得的实参隶属适当类型时才能通过编译。



member function templates（成员函数模板）的作用不限于构造函数，它也能去实现赋值操作。

std::shared_ptr 的前身 tr1::shared_ptr 的一份摘录如下：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=N2Y5ZTgwYzE1N2ZlN2U0Y2U2MDFkMjNkODA0ZmQ2ZjhfbFg4UlhzWll3YnZZQVFNSGUxRzlsSFpYTjJ6czM4MmhfVG9rZW46Ym94azR5NHVWU0Z1MVFnenA0MThVTWE0b3FnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZWUyYTc0ZjM0NTZjNWJhZjYzZTgzMGRiYzU0MTgyYTFfZ3ZOeTdvRFo5eHlBVWQ5djFWc2o5MXZ5WEZ4azBZOEpfVG9rZW46Ym94azRIMG9EcjNWUmdsV1ROdUZ5czhKTGJnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述所有构造函数都是 explicit，“泛化 copy 构造函数除外”，意味着从某个 shared_ptr 类型隐式转换至另一个 shared_ptr 类型是被允许的。



成员函数模板并不改变语言的基本规则，如果程序需要一个 copy 构造函数，但我们没有声明它，那编译器会暗自生成一个。在 class 内声明泛化 copy 构造函数并不会阻止编译器生成它们自己的 copy 构造函数。对于构造函数，如果有需要，我们可以即声明一个普通构造函数（non-template），和一个泛化 copy 构造函数。下面是 tr1::shared_ptr 的一份定义摘要：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MTczNzZiYzIyYjUyMGQxNDdlOTE0MTVhZWFkYzJhZGRfMjNRZVNpYXpUd1huWjFZUG9GeDhzQkdtb2NERzNkdTJfVG9rZW46Ym94azRrdnV2YUpCUVZuM0J6OVRXRWhBV1VnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

#### 总结

- 使用成员模板函数（member function templates） 生成“可接受所有兼容类型”的函数。
- 如果我们声明 member template 用于“泛化 copy 构造”或“泛化 assigment 操作”，我们还是需要声明正常的 copy 构造函数 和 copy assignment 操作符。



### 条款 46：需要类型转换时请为模板定义非成员函数

条款 24 讨论了只有 non-member 函数才有能力“在所有实参身上实施隐式类型转换”，该条款以 Rational class 的 operator* 函数为例，本条款首先以看似无害的改动扩充 条款 24 的讨论：本条款将 Rational 和 operator* 模板化了：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NWVkZDY2YWIzNjQ2YThlMDM3YTY5ZDRlYmE2MTY0MmFfT0VKVVVldGtic2RPZm9SSnlua3BEUndQSmFwNUVudmFfVG9rZW46Ym94azR2Nm5MVjlXcm52RXUzNGpaZ1NIY1doXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

就像条款 24 一样，我们希望它支持混合式算术运算，所以我们希望以下代码顺利通过编译：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MGVlNTRjZDA5M2JhYTQxNmY1YTI2NjNiNDc1N2Y2MDZfdEpGdUh6Zk5wSUVKZGhCdmFUWTRESURPa055bVRuYUNfVG9rZW46Ym94azRWV0xobDgyQ05OdWNDSDR4b1RVT2tlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述代码给我们的启示是，模板化的 Rational 内的某些东西似乎和其 non-template 版本不同。因为在这里编译器不知道我们想调用哪个函数。

取而代之的是，它们试图想出什么函数被命名为 operator* 的 template 具现化出来。它们知道它们应该可以具现化某个“名为 operator* 并接受两个 Rational<T>参数”的函数，但为完成这个一具现化行动，必须知道 T 是什么。问题是它们没有这个能耐。（它们代指编译器）。

```
Rational<int> result = oneHalf * 2;
```

为了推导 T，编译器看了看 operator* 调用动作的实参类型，本例子中那些类型分别是：Rational<int>（oneHalf的类型） 和 int (2 的类型)。

对于 oneHalf 来说并不困难，从它的定义式可知它的 T 类型为 int。那对于第二实参 int，编译器如何根据这个推算出 T。我们可能会期盼编译器使用 Rational<int> 的 non-explicit 构造函数将 2 转换为 Rational<int>，进而将 T 推导为 int。但编译器不这么做，因为 template 实参推导过程从不将隐式类型转换函数纳入考虑。这样的转换在函数调用过程中的确被使用，但在调用一个函数之前，首先必须知道那个函数存在。而为了知道它，必须先为相关的 function template 推导出参数类型。然而在 template 实参推导过程中并不考虑采纳“通过构造函数而发生的”隐式类型转换。

现在我们处于 template part of C++ 领域内，template 实参推导是我们的重大议题。



只要利用一个事实，我们可以缓和编译器在 template 实参推导方面受到的挑战：template class  内的 friend 声明式可以表示某个特定函数。意味着 class Rational<T> 可以声明 operator* 是它的一个 friend 函数。class template 并不依赖 template 实参推导（后者只施行于 function templates 身上），所以编译器总是能够在 class Rational<T> 具现化时得知 T。

因此，令 Rational<T> class 声明适当的 operator* 为其 friend 函数，可简化整个问题。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NDA4ZWYxYjdkZGI5OWI5NTdkYjQ5Y2UyY2MwODU3MzNfVUhNWjZ5N0N3WmN0a2xNcjRSQ2hTS0dJd2dSZXpxQzJfVG9rZW46Ym94azRhOGpLQnlVVjJwZTlqcENxTUNaUjRkXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MjMzOTU4OGQxZjRjNjU0MThiNGRjM2I0YTEyMWJhNWRfOG1VVWpQU1ZBdTlqamN3ejE4YW1JeXlMYWZIczFSSUtfVG9rZW46Ym94azR2dE1WTnVQaWJoelZmMnBnTnJUWjNjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这样，friend 函数 operator*（接受 Rational<int>参数）也就被声明出来，它现在是一个函数，而非函数模板，因此编译器可以在调用它时使用隐式类型转换（例如 Rational 的 non-explicit 构造函数），而这便是混合式调用（ `oneHalf * 2`） 之所以成功的原因。

本例中的 operator* 被声明为接受并返回 Rational（而非 Rational<T>&），它其实等价于以下形式：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZmVhOWE2ZmRlMTZkODljZmM2MWMyOTFlNDE5MDMyOTNfbzZra3d0WlJuTEI2VEhmSU8yaEp0QXV2R2FxaWVKS2lfVG9rZW46Ym94azRIRkhVYWVwVzRVMG44aUhRZ1gxNGw2XzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述代码虽然能通过编译，但无法运行，会出现连接错误。因为意图让此 class 外部的 operator* template 为 class 内声明的 operator* 提供定义式。这是行不通的，如果我们自己声明了一个函数，就有责任定义那个函数。如果没有提供定义式，连接器当然找不到它。

最简单可行的方法是将 operator* 函数本体合并至其声明式内：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NzcxYWQzZGQ1N2FkNzI3MGNiYTQ0MDAwOTZjY2NhZDVfd3kyOTduZ0I1WTdnNXBqdEE2ZWJiWEJ3anl0eHoxbzdfVG9rZW46Ym94azRrVkVhblh4blhzRHBWSFpsUGpnNUxlXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这项技术有趣的一点是，我们虽然使用 friend，却与 friend 的传统用途“访问 class 的 non-public 成分”毫不相关。为了让类型转换可能发生于所有实参身上，我们需要一个 non-member 函数（条款 24），为了令这个函数被自动具现化，我们需要将它声明在 class 内部。



而在 class 内部声明 non-member 函数的唯一办法就是：令他成为一个 friend。



条款30 说过，定义于 class 内部的函数都暗自成为 inline，包括像 operator* 这样的 friend 函数。为了将这样的 inline 声明所带来的冲击最小化，做法是令 operator* 不做任何事，只调用一个定义于 class 外部的辅助函数。

对于本例而言，这样做并没有太大意义。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=OWY4ZjNkMmQzYWY3YWMxMjFmNjNjZTU1MWRhMDEwOGZfRlNuUmswNEx4UnJiRENldWdzTzJ4RlFDZEJpamQ4R0hfVG9rZW46Ym94azR3T0s1VnNYcDdFTVNNUTZQSGx3c3FxXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NzVkN2ExZWRmZWNiZTdiNDg4NzQyMTdlZDFiZTc0ZjZfVkdNYnN0c0VrenhiZ2FKdGJtVGRhYmsyNFVreEU2aEFfVG9rZW46Ym94azRrUEY0OVRUVzJrN3ZobGR4a3Zib2NmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

作为一个 template，doMultiply 当然不支持混合式乘法，但它其实也不需要，它只被 operator* 调用，而 operator* 支持了混合式操作。本质上 operator* 支持了类型转换所需的任何东西。



#### 总结

当我们编写一个 class template，而它所提供的“与此 template 相关的” 函数支持“所有参数的隐式类型转换”时，请将那些函数定义为“class template 内部的friend 函数”。





### 条款47：请使用 traits classes 表现类型信息

#### 47.1 迭代器的分类

STL 中共有 5 种迭代器，对应于它们支持的操作：

1. input 迭代器：只能向前移动，一次一步，客户只可读它们所指的东西，而且只能读取一次。它们模仿指向输入文件的阅读指针（read pointer）。例如 C++ 标准库中的：istream_iterator
2. output 迭代器：只能向前移动，一次一步，客户只可写它们所指的东西，而且只可写一次。它们模仿指向输入文件的写指针（write pointer）。例如 C++ 标准库的：ostream_iterator
3. forward 迭代器：可以做前述两种分类的每一件事，而且可以读或写所指物一次以上。
4. Bidirectional 迭代器比上一个分类更强大，它除了可以向前移动，还可以向后移动。例如：STL 的 list，set，multiset，map 和 multimap 的迭代器。
5. random access 迭代器：最强大的迭代器，除了具有上一个分类的所有功能，它还可以进行“迭代器算术”，即它可以在常量时间向前或向后跳跃任意距离。它类似于指针。例如：vector, deque 和 string 提供的迭代器属于这一分类。

对于这 5 种迭代器，C++ 标准程序库分别提供专属的 tag struct 加以表示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MWY2NzU2NDU3YzZjOTk5MGI0ZTQ0MDQyMDllMmI2YjNfbTltWHRCRXZGcjFIbnZ3YTU3a29LNVNnS05tdnpPczlfVG9rZW46Ym94azR5Y01uakZES0x6cjJZQUx2NGlmbHExXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这些 struct 之间的继承关系是有效的 is-a 关系。所有的 forward 迭代器都是 input 迭代器。



#### 47.2 std::advance 的实现 

STL 中有若干工具性质的 template，其中一个名为 advance，用来将某个迭代器移动某个给定举例：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=Zjg0MTI2NDUzNTE4MmJlMzVkZjkwOGJhMGM4MTI1N2JfVUlOWjNwaWlvdWpWMko2YThWRDV1dmhIc2lEVnJSQ0pfVG9rZW46Ym94azRiakcxMDZBOHp4WHAwTFZaRmlEVFFjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

表面上看 advance 只是做 iter+=d 动作，但其实只有 random access 迭代器才支持 += 操作，对于其他迭代器，advance 必须反复施行 ++ 或 -- 共 d 次。

为了充分利用各种迭代器的优势，我们这样设计 advance：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ODJiZTc4ZjU2Y2Q1NWRmMTk1ZDhjMGJiNGI1Y2VkOTNfcDlHUVNaektkTWdwT3lmejdZN2RpWjlpUlBuWnp5ODJfVG9rZW46Ym94azRZMFJKeWpHMndQUTVmMDlPUWFNcmpnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这种做法必须判断 iter 是否为 random access 迭代器，即我们需要取得类型的某些信息，这就用到了 trait：它允许我们在编译期间取得某些类型信息。

Trait 并不是 C++ 关键字或某个预定义的构件：它们是一种技术，也是一个 C++ 程序员共同遵守的协议。它的要求之一是：它对内置类型和用户自定义类型的表现必须一样好。

Trait 必须能够用于内置类型，意味着“类型内的嵌套信息”这种东西出局了，因为我们无法将信息嵌套于原始指针内，因此类型的 trait 信息必须位于类型自身之外。标准技术把它放进一个 template 即其一或多个特化版本种。这样的 template 在标准程序库种有多个，其中针对迭代器的被命名为 iterator_traits：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MTdiYzY3Y2E5YzEyOTM3MWUzZmZiZGVhOTMyYTkzZThfNTN2NVhBcVc3Z2JsdUlneWJhVXRBNVF1ZkNZYmVTcmtfVG9rZW46Ym94azRVMUp0SFpTMnlaZ1JXUVptdHFxd0llXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

iterator_traits 的运作方式，针对每一个类型 IterT ，在 struct iterator_traits<IterT>内一定声明某个 typedef 名为 iterator_category。这个 typedef 用来确认 IerT 的迭代器分类：

#### 47.3 iterator_traits的两种实现

##### 针对自定义类型

首先它要求每一个“用户自定义的迭代器类型”必须嵌套一个 typedef，名为 iterator_category。例如：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MWMzOTcyNWM0Nzc1ZDdiNjJjNmNhMTZkMThkYzRjOWVfWHB4NjFEZnJmSjYyVzE5VzN5eWNEbWhPWHd1cmFGQUhfVG9rZW46Ym94azRhbmlueGNvaXRWWDc0ZzdJU2pwam5nXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

这对用户自定义类型行得通，但对指针行不通，因为指针不可能嵌套 typedef。iterator_traits 的第二部分如下，用来对付指针:

##### 针对内置类型

为了支持指针迭代器， iterator_traits 特别针对指针类型提供了一个偏特化版本。例如：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZmQ4ZGI1YTA0YThhOTAyZWJmY2NlZDljMGJhN2Y0YWFfeUNTSk1hRHRSTmVSbm5aVXNRaGRjYk1QVHpkcUoyTm9fVG9rZW46Ym94azRDVDV5QVZpaTZwa2tCMnJuUnFoT1JpXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



现在我们知道怎么设计一个 traits class 了：

- 确认可取得的类型相关信息，例如对迭代器而言，希望将来可取得其分类。
- 为该信息选择一个名称（例如 iterator_category）
- 提供一个 template 和一组特化版本（上述的 iterator_traits），内含你希望支持的类型相关信息。



现在有了 iterator_traits，我们可以对 advance 实践先前的伪代码：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NDVkM2EwOGVkMzc0YWZkNDhmNTEyNGYxY2VlMDFhZjZfVzZHbFVSVEd5RUdzdnowcGd6VVFIWU9VV0pIbWRoM2FfVG9rZW46Ym94azRiUlllYWY2ZGx4YU51cE5FeHNjcGFmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

其实上述代码会导致编译问题，IterT 类型可以在编译器知道，但 if 语句却是在运行期才确定。

我们真正想要的是一个条件式，也就是 if...else 语句，判断“编译器确定的类型”。利用 C++ 的重载可以实现它。

当我们重载某个函数 f，必须详细叙述各个重载件的参数类型。当我们调用 f，编译器便根据传来的实参选择最适当的重载件。这正是一个针对类型而发生的“编译器条件式”。为了实现这个设想，我们定义一个函数 doAdvance 来作为被重载函数：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NzAwNTM2Njk4ZjFmYmNiNjc2Y2NkODdjMjc5MjdmMjRfZXRvM01BVGgyMWlLMEJUeEhwWHlNYVVEN09vQTc4anNfVG9rZW46Ym94azRjOXJwaXQ2bVhVWUpUd2ExUW42NjlmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



有了这些 doAdvance 重载版本，advance 需要做的就是调用它们并额外传递一个对象，后者必须带有适当的迭代器分类。于是编译器运用重载解析机制调用适当的实现代码：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NGY4OGE5NDc2NmYzNzIyNDViZjY4NDU3ODI1ZjUxMzVfc0JtM1FsQ2hXMGNiTTdqcHQwY2ozbEp1akpmUXFna2dfVG9rZW46Ym94azQ4OWFMcnBoSVNnT0xOTzBlcTRlNm9jXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

#### 47.4 如何使用一个 traits class 

- 建立一组重载函数或函数模板（例如 doAdvance）,彼此间的差异只在各自的 traits 参数
- 建立一个控制函数或函数模板（例如 advance），它调用上述那些函数（doAdvance）并传递 traits class 所提供的信息。



Traits 广泛运用于标准库，其中包括 iterator_traits，它除了提供 iterator_category ，还提供另四份迭代器相关信息（比如 value_type，见条款 42）。

除此之外，还有 char_traits 用来保存字符类型的相关信息，以及 numeric_limits 用来保存数值类型的相关信息，例如某数值类型可表现的最小值和最大值。



std 还包括许多 traits classes 用以提供类型相关信息，包括 is_fundamental<T> （判断 T 是否为内置类型），is_array<T>（判断 T 是否为数组类型），以及 is_base_of<T1, T2>（T1 和 T2相同，或者 T1 是 T2 的 base class）



#### 47.5 总结

Traits class 使得“类型相关信息”在编译器可用。它们以 templates 和 “templates 特化”完成实现。

整合重载技术后， traits classes 有可能在编译器对类型执行 if...else 测试



### 条款 48：认识 template 元编程

Template metaprogramming（TMP，模板元编程）是编写 template-based C++ 程序并执行于编译期的过程。

所谓 template 元程序，就是 C++ 写成，执行于 C++ 编译器内的程序。

TMP 是于 1990s 初期被发现，后被证明十分有用。

TMP 有以下优点:

1. 它让某些事情更容易，如果没有它，那些事情将是困难的，甚至不可能的。
2. template metaprogram 执行于 C++ 编译期，因此可以将工作从运行期转移到编译期。
3. 使用 TMP 的 C++ 程序可能在每一方面都更高效：较小的可执行文件，较短的运行时间，较少的内存需要。

缺点是：编译时间变长了。



考虑 条款 47 的 STL advance 伪代码：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MmNlYWVmZTNhYTk2NDljMWVkYjI3MTU2Y2IwNWQ4Y2VfWjlOYmVEVVdhYWdHZUdtOUJIeDk4bHBRSVdDSUY3RGlfVG9rZW46Ym94azQwWllvaG9GcnNVc2VmUDBsSEdQTVJoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



我们可以使用 typeid 来实现上述伪代码。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MDNhODU0NGE4MDZhNTE2Mjg0YmNiZmMxZWQyM2QwYjBfa3JBQk1xR1hScjFrQnIxMmRxMkd6RjdNSHFScHFmY3pfVG9rZW46Ym94azRRQWl5NEJ3V09RNnFwU1E0aElRTVpiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

在此方案种：

1. 类型测试发生于运行期而非编译期。
2. “运行期类型测试”代码会出现在可执行文件中。



为了支持下述调用：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=MzcyYzdjOTM4ODgwYWNjYjIyY2ZmNmNlMjBiZjU4MDhfbEhQTm44UGJ6VjlvZFlWVDVkanU4SFZqVTM5RjI0cUJfVG9rZW46Ym94azRiQXpoV1hFTjRjTmU2UWVFUWV0NVJjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

我们这样写 advance：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YjYxMGY1ZjRmNTgwNmYyMjExMGM0ZTQ3NjljNmYyNjZfUXZsV3lROUJmaHdGTFFCSjNoYVNwSnFmWEFMOTg3UUVfVG9rZW46Ym94azRzOUNSMTV5OXBwZUkzb3BwdzdSSjZiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述代码存在的缺陷：list<int>::iterator 并不是 random access 迭代器，它并不支持 +=，所以代码中的 if 语句从不会被执行，但编译器必须确保所有代码都有效，纵使是不会执行起来的代码。与此对比的是 traits-base TMP 解法，其针对不同类型而进行的代码，被拆分为不同的函数，每个函数所使用的操作都可以用于该函数所对付的类型。



TMP 已被证明是个“图灵完全”机器，意思是它的威力大到可以计算任何事物。使用 TMP 可以声明变量，执行循环，编写及调用函数。

条款 47 所展示的 TMP is...else 条件句由 template 和其特化表现出来，不过那是汇编语言层级的 TMP，针对 TMP 而设计的程序库（例如 Boost's MPL）提供了更高层级的语法。



TMP 中的循环由递归实现，它并不涉及递归函数调用，而是涉及“递归模板具现化”。

编译期计算阶乘可以算是 TMP 的 Hello world 程序，该程序示范如何通过“递归模板具现化”实现循环，以及如何在 TMP 中创建和使用变量：

 

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YzIyOTNkNjFlZmUzNmJlYmIxNmYxNzlhOGU3ZGRjYzNfaVQxek9GNkptRTUyVVFzWTNIYkJUamNqZ1VLOXhyQTZfVG9rZW46Ym94azRvRFhGaXdMVDRQNHZmN2czRG5iOWljXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

调用 Factorial<n>::value 就可以得到 n 阶乘值，但 n 必须是编译期常量。

循环发生在 template 具现体 Factorial<n> 内部使用另一个 template 具现体 factorial<n-1>时。和所有递归调用一样，我们需要一个特殊情况来结束递归，这个便是 template 特化体 Factorial<0>。

每个 Factorial template 具现体都是一个 struct，每个 struct 都使用 enum hack 声明一个名为 value 的 TMP 变量，value 用来保存当前计算所得的阶乘值。由于 TMP 以“递归模板具现化”取代循环，每个具现体有自己的一份 value，而每个 value 有其循环内的适当值。



TMP 可以达成什么目标：

1. 确保量度单位正确。使用 TMP 可以确保在编译期程序中所有度量单位的组合都正确，不论其计算多么复杂。
2. 优化矩阵运行：条款 21 提到过的 operator* 函数必须返回新对象，条款 44 又导入了一个 SquareMatrix class。例如以下代码：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZTZiNjRmZGRjNzI1YjJjNDU5MjNjNDFhNDc1ZGNmY2FfczU4aFRqcHQwZnpVNlBPbXVvdTBoc0pIc09YVmRnZE1fVG9rZW46Ym94azRDV0duWGJuWE9VYkNveUo4ekFDRm03XzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

以“正常的”函数调用动作计算 result，会创建 4 个暂时性矩阵，每一个用来存储 operator* 的调用结果。如果使用高级的 TMP 相关的 template 技术，即所谓的 expression templates ，就有可能消除那些灵时对象并合并循环。于是 TMP 软件使用较少的内存，执行速度更快。

1. 可以生成客户定制的涉及模式实现品，设计模式如 Strategy，Observer，Visitor 等等都可以多种方式实现出来。



#### 总结

TMP 将工作由运行期移往编译期，可以实现早期错误侦测和更高的执行效率。

TMP 可被用来生成“基于政策选择组合”的客户定制代码。

我的理解如下：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZTcwNmEyZGZjOTJiMmY5NmIyMTBiMGY2M2Q4NDY3NDFfbEpBcjkxOXN3ek9lZWdhZmNTRnNGaW9nMG81V2VVY3lfVG9rZW46Ym94azRoQ2lEN1ZZbWV5RlZ2NDNqMmFWZXdVXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)





## 八、定制 new 和 delete

本章的重点在于：operator new 和 operator delete，配角是 new-handler，当 operator new 无法满足客户的内存需求时所调用的函数。



### 条款 49：了解 new-handler 的行为

当 operator new 无法分配足够内存时，它会抛出异常，在以前它会返回一个 null 指针。当 operator new  抛出异常之前，它会先调用一个客户指定的错误处理函数，名叫 new_handler 的全局函数，为了指定该函数，客户必须调用 set_new_handler，该函数是声明于 <new> 的一个标准库程序。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZWZlNTU3YzAzMTBlNzc0MzJlMDlmMWU4Mjk2YTI4YzJfeTNkTG8xTEJaQTVEWjFMV2Y1dWk3ZVJIMVpUTWlmUk5fVG9rZW46Ym94azRVRm56MGROVFZZMlBpNXlTWDloVXZoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

new_handler 是个函数指针，set_new_handler 则是“获得一个new_handler 并返回一个 new_handler”，获得的 new_handler 用于设置当前的 new_handler 函数，返回之前的 new_handler 函数。



我们可以这样使用它们：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YzUzOTMxNmZkMWQ2M2Q5NzE4MzkxNGUxNGMxNjFmMTNfTFlmY3dvNTFFTEZYTVVJb1ZaNTI2NUgxTEt5YTdmOVRfVG9rZW46Ym94azRjYXNCdXZhN0YxNERaMUNaMnY5TUdmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZTY1Njk0ZjY4NjliMjk0Mjg4YzA3NzAwNjA3ODgwODJfenVWYUd6VG4xQzZkRVFMUXVGd3dQaU90Rmc3RUozVHZfVG9rZW46Ym94azRRVVJjc0J2bzA3ckpPQXhydmtLbUhnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

当 operator new 无法分配足够的空间时， outOfMem 会被调用。

一个设计良好的 new-handler 函数必须做以下事情：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=YWUyYmM5MWQyOTEzYTA5MDUzYzUzNWRjMGJlZjY3NDNfZWI1Q2hCM0ttR1FNb1BKNFhUYWZSQVE2ZkJqMVpwc1JfVG9rZW46Ym94azRmRlM3d1k0MjVCT0NWc2d1cGp1eTZiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



#### 模拟 class 专属 new_handler

如下图所示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZjI5YjE2OGFkNzhjM2Y3YTk0OWQwZDVlMTUyNWJkZGVfQ0h6WVVrb2RIdTBuR0ppd3BWTWhYeUhhbXBMbmlnYmJfVG9rZW46Ym94azRwbDc5UU1UNkVrQ3RyWFlrU09IWmRkXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

C++ 并不支持 class 专属的 new-handler，但可以自己实现这种行为。

例如：我们打算处理 Widget class 的内存分配失败的情况，如下所示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ODY4YzE0Njk0ZWI2YTA3ZDg2MjU2ODczNzlmOGZjODhfOHVBck5sa1R6eVQ1VXZmdnVZekxhcEdqSGpBWFpFZmlfVG9rZW46Ym94azRqNVlsWk5lckZwUTRmVWE0cmt0OHhmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

关于上述代码有一点需要注意，一般情况下 static 成员必须在 class 的定义式之外被定义。

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NWVmNDk2MGI3M2E1MjgyNDA4Yjk4ZTcyNmIxOGY0MDhfbjg0Q21iWVVKM2dWTzdCUGo4Y0JHS3NJcnNJc2RWRmxfVG9rZW46Ym94azRGTW5OcExubU1HdnVQZERneXZEZzZiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

Widget 内的 set_new_handler 函数将它获得的指针存储起来，然后返回先前存储的指针。这也正是 std::set_new_handler 的做法：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ODljMjIyM2U5NjI3Y2RmYTk2NWM1ZmM2YzQzOGJhYWJfaXJsNUJMZGFCR1g0WGFFZzdNRTMwOVhVaHNvNWtDcjRfVG9rZW46Ym94azRaTTNaeTBMMHY1RG1wQnhheFpaaEhkXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

此处的 new_handler 指针需要作为资源处理，即在调用特定对象的 new 时设置 new_handler 指针，使用完之后需要恢复以前的 new_handler 指针。为此我们使用 RAII 来管理 new_handler 。如下所示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=Mzg1ODg4NGVjODhmZGVlYmE0MzM2MDZjZGY2NGJlNmFfWlZPUFVDRnA1WjlsU0dnMWRTeWFEbE5MbzlHYjh2UUFfVG9rZW46Ym94azQxUFRnbmZkekdDeEtpa2s0ZzNBM21nXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



现在 Widget::operator new 的实现就比较简单了：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=OGExZjU1YmFiZDgyYTE3YWM0ZmQ0MTRlZjYxOTExMWFfcEIySVlPYWpWZGxucWxhdXNtSnBkbnhXSWdJVlV5Q1JfVG9rZW46Ym94azQ5VnByMm02UlRFcXdrMXZ0RTQ0NlZiXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



Widget 的客户可以如下调用：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NWJlM2RlYTVlMDhhYjViYzAzN2ExMGNmNzEyMjc0ZWZfY3hRZTJPU3hkdFZVVVpManRKa01Xd2NHb0RYYWNFZ1lfVG9rZW46Ym94azRhcVBZSkRnaVJxaFNWRjdjQThBM2hnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

#### 模板化

以上代码只对 Widget 有效，我们可以将上述代码模板化，这样就可以用相同的代码支持不同的自定义类型。这里暂略，请参考《Effective C++ 第三版》245页。



#### 总结

- set_new_handler 允许客户指定一个函数，在内存分配无法满足时被调用。



### 条款 50：了解 new 和 delete 的合理替换时机

什么时候我们可以替换编译器提供的 operator new 或 operator delete 呢？下面给出三个最常见理由：

1. 用来检测运用上的错误：例如，一个很常见的错误是 "overruns"（写入点在分配区块尾端之后）或 "underruns"（写入点在分配区块值之前），我们可以分配额外的空间，例如在所分配内存的前后各放置一个签名，然后在 operator delete 中检查两个签名是否被改写，就可以判断是哪个指针犯了上述错误。
2. 强化效能。编译器所提供的 operator new 和 operator delete 是为了各种场合而生，所以它不会在任何场景下都百分百高效，我们可以针对特定的场合替换 operator new 和 operator delete 会提高代码运行效率。
3. 收集使用上的统计数据。为了定制 new 和 delete，我们需要确定分配区块的大小，以及分配策略（先进先出，后进后出等等）。为了收集上述信息，我们可以用定制型的 new 和 delete 收集这些信息。

例如，下面的代码可以检测 "overruns"（写入点在分配区块尾端之后）或 "underruns"（写入点在分配区块值之前）错误，该实现还不完善，稍后指出：



![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=OWI5MTg1MDJiNjNjMDcyODg5MmRiMTA3MDE5ZDY2NzNfdHFUSkx5NGpzdGpBSnJ0elhHTjUyOEd4ZWdxT09ZZWFfVG9rZW46Ym94azQ4RUNxY2N2UGx1ZkZSMEM3RjBXWEpoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



上述代码存在的问题：

- 并未考虑内存对齐问题：许多计算机体系结构要求特定类型必须放在特定的地址上。例如：32 位系统可能要求指针的地址必须是 4 的倍数（four-byte aligned）。如果没有奉行这个约定，可能会导致硬件异常，或降低内存寻址效率（X86 架构）。malloc 就是以这种要求工作的，返回一个来自 malloc 的指针是安全的，但上述代码返回的是来自 malloc 且偏移一个 int 大小的指针，没人能保证它的安全。
- 条款 51 会说明，operator new 应该内含一个循环，反复调用某个 new-handling 函数，这里没有。



#### 总结

替换缺省的 new 和 delete 有以下优点：

1. 检测运用错误。
2. 收集动态分配内存的使用信息，
3. 增加分配和归还的速度。
4. 降低缺省内存管理器带来的空间额外开销。（针对小型对象而开发的分配器，例如 Boost 的 Pool 程序库）本质上消除了这样的额外开销。
5. 弥补缺省分配器中的非最佳齐位。例如：x86 体系，double 的访问速度最快是 8-byte 对齐。但编译器自带的 operator new 并不保证对动态分配而得的 double 采用 8-byte 对齐。
6. 将相关对象集中管理：如果某个数据结构往往被一起使用，如果为此数据结构创建另一个 heap，可以降低“内存页错误（page faults）”的频率。
7. 获得非传统的行为：例如将归还的内存初始化为 0，增加应用程序的安全性。

#### 

### 条款 51：编写 new 和 delete 时需要固守常规

自定义 operator new 需要遵守以下规则：

1. 返回正确的值，内存不足时必须调用 new-handling 函数
2. 必须对付零内存需要。



operator new 返回一个指针指向那个内存，内存不足时抛出一个 bad_alloc 异常。不过，在分配失败时，它会反复调用 new-handling 函数，只有当指向 new-handlig 函数的指针是 null，operator new 才会抛出异常。

C++ 规定，即使客户要求 0 bytes，operator new 也得返回一个合法的指针，如下列代码所示：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZmNmNzgzZWZjOTZlZmJhNDYwZTJjZDlmMTgzNWFkMjhfUDF5RHJLNjZZbXlYclpHeTdBakJMYUNmTXJjTTM1SGRfVG9rZW46Ym94azR3VXRCVldkTXMySVBadGVxUU1FSlNjXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NjFjMDIyMzAyNGYxZTlkZmE3NDY4MzJkYzhkMzg0MDVfSzJocm42MnlQeXFoTlI1enNtdGJPMHhsUXlHQXRrc1hfVG9rZW46Ym94azR2dWxrVHFhaTc1R3N3Mk52akRYSW9lXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



#### 当涉及到继承时

##### new

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NDFjNGUyOTgzZmQ0MmFhNDAzYTk0OGY5ODA4Mzg3ZTNfaFZhZVBHQUpJZ0hKV0JTbUk2djh4VHZ1NXlVaUZmb2dfVG9rZW46Ym94azREbkRpWThSRUdaM0kyOGkzek1KSm5oXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

派生类指针，却调用 Base::operator new处理此问题的最佳做法如下：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=M2NmN2U3ODg0OTI1Nzc2Mjk0NzI1YzdiOTkzMGFkYjFfQXBObzZaa1ZCVlZZSzRuRzJzOWJDcXhOWjU2ZDlxdU9fVG9rZW46Ym94azRMYlZWdzZuM2hJRWlQVmVaNGdkeVZqXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



##### delete

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZGE5MjcyZjhhMzYxYTBiMDMwYzZjMDZlMWVkMWM3MWNfRzBzQ3NEbDZEa0x4NktQNGU0Ymo2cnMxTjBLNU1UdUhfVG9rZW46Ym94azQ4ZjBOejBPRmdPRzZhWWd5eVo1cWFkXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)





如果被删除的是 null 指针，则不做任何事情：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=Zjc4M2U1NGQ3NDk1MjI0YzE4Y2YyMGFmODYyNTNkNzdfdWo0Q1k1MUNBNElmSWdTT0ltNDAxMldHNHVMaEQzYXpfVG9rZW46Ym94azR3eDBCblpycExmZVZTZ3I1ZFpzS2toXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



#### 总结

- operator new 应该内含一个无限循环，并在其中尝试分配内存，如果它无法满足需求，就该调用 new-handler。它也应该能处理 0 bytes 申请。class 专属版本则还应该处理“比正确大小更大的（错误）申请”
- operator delete 应该在收到 null 指针时不做任何事。class 专属版本还应该处理“比正确大小更大的（错误申请）”。



### 条款 52：写了 placement new 也要写 placemen delete

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ODMyNTY1ZTVlMGIzYjEyY2E4YjJhNDcxOGVjYWI1ZTZfVTdzelhiQmljNnA0VElXSjlJZnpnbVJjQkh5NGI1dXZfVG9rZW46Ym94azRFY1pSemxMMHhJN2F0RTlCV2pld2lnXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

上述代码有两个步骤，一个是用于分配内存的 operator new，一个是 Widget 的 default 构造函数。假设第一个调用成功，第二个抛出异常，步骤一分配的内存必须被释放，否则会造成内存泄漏。

完成上述任务的责任落在 C++ 运行期系统上。运行期系统会高高兴兴地调用步骤一所调用的 operator new 所对应的 operator delete 版本，前提是目期拥有正常签名式的 new 和 delete ：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NjE3YzRiOWE4ODAzMmZiMTVkZjE0N2NhNWRmNmVhYmZfZHBRcFBxQXNwTlV3VVJweXB0TjJBZzBReDBLQ09xcHNfVG9rZW46Ym94azRaY2lOd3lPWndSdGtSTEl2SHpCdnVoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ODdlMTAyMDhjMThlZGY1MDA2NzEwYmU5YjY1OTQ2ODdfZkQxZFNPYVBwcEJkdm9icDU1emEySE5UYzFqRkY5V1ZfVG9rZW46Ym94azRCSnl0UEJWTkVEd09HTGFTcHlBejJmXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

命名遮盖问题

凡是想以自定形式扩充标准形式的客户，可利用继承机制及 using 声明式取得标准形式：

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=ZGUzMTE4M2EwY2YyMmM5MGFhYjRlNjBkOWRjZTc5NTlfdUZ3NUJTZ2NRWnVlRWlRTHVMMURQTEN6REhxbFJob21fVG9rZW46Ym94azRPNG1TMkFVMHRMeWpuOUdKdFBZNUFkXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)



#### 总结

![img](https://xiaomi.f.mioffice.cn/space/api/box/stream/download/asynccode/?code=NDk0ZjM1MmUzOTMxMjM1ZWFhNzFlNWI0MjgxODYyYjNfaks5eWMzV1hUbVB1dHZjZGZLd01FUHRtUlJLRVhmN1ZfVG9rZW46Ym94azRVMXpzSDVSWkYxVURaamE4azRLbXZoXzE2NzM1MTg5NTA6MTY3MzUyMjU1MF9WNA)

