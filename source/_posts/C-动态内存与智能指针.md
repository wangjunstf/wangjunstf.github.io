---
title: C++动态内存与智能指针
date: 2021-09-12 10:35:29
updated:
categories: [CPP]
tags: [CPP,动态内存,智能指针]
--- 

使用智能指针需包含以下头文件：`#include  <memory>` 
## shared_ptr类
shared_ptr允许多个指针指向同一个对象
**支持的操作：**
<!-- more -->
```c++
shared_ptr<T> sp;      //空智能指针，可以指向类型为T的对象
p                      //将p用作一个条件判断，若p指向一个对象，则为true
*p                     //解引用sp, 获得它指向的对象
p->mem                 //等价于(*p).mem
p.get()                //返回p中保存的指针。要小心使用，若指针指针释放了其对象，返回的指针所指向的对象也就消失了
swap(p,q) 等价于 p.swap(q)              //交换p和q中的指针
make_shared<T>(args)   //返回一个shared_ptr，指向一个动态分配的类型为T的对象。使用args初始化次对象
shared_ptr<T>p(q)       //p是shared_ptr的拷贝；次操作会递增q中的计数器。q中的指针必须能转换为T*
p=q                     //p和q都是shared_ptr，所保存的指针必须能相互转换。此操作会递减p的引用计数，递增q的引用计数；若p的引用计数变为0, 则其管理的原内存会被释放
p.unique()              //若p.use_count()为1，返回true; 否则返回false
p.use_count()           // 返回与p共享对象的智能指针的数量；可能很慢，主要用于调试

```



### shared_ptr和new结合使用

```c++
shared_ptr<double> p1(new int(1024));   //p1指向一个值为42的int
shared_ptr<double> p1 = new int(1024);  //错误，必须使用直接初始化形式
```



### 改变shared_ptr的方法

```c++
shared_ptr<T>p(u)    //p从unique_ptr u那里接管了对象的所有权：将u置为空
shared_ptr<T>p(q,d)  //p接管了内置指针q所指向的对象的所有权。q必须能转换为T*类型。p将使用可调用对象d来代替delete
p.reset()            //若p为唯一指向其对象的shared_ptr，reset会释放此对象
p.reset(q)           //释放为p，令p指向q
p.reset(q,d)         //调用d而不是delete来释放q
  
```



### 自定义释放操作

```c++
// 定义自己的释放函数
void end_connection(connection *p) {disconnect(*p);}   

void f(destination &d){
  connection c = connect(&d);
  shared_ptr<connection> p(&c, end_connection);   //当f退出时（即使是由于异常而退出），connection会被正确关闭
}
```



## unique_ptr类

与shared_ptr不一样，某个时刻只能有一个unique_ptr指向一个给定对象。

```c++
unique_ptr<int> p(new int(42));  //unique_ptr必须采用直接初始化形式，而不能采用拷贝和赋值
```

### unique_ptr的常见操作

```c++
unique_ptr<T> u1;        //空unique_ptr
unique_ptr<T,D>u2;       //空unique_ptr，u2使用类型为D的可调用对象来释放它的指针
unique_ptr<T,D>u(d);     //空unique_ptr，指向类型为T的对象，用类型为D的对象d代替delete
u = nullptr              //释放u指向的对象，将u置为空
u.release()              //u放弃对指针的控制权，返回指针，将u置为空
u.reset()                //将u置为空
u.reset(q)               //将u置为空，令u指向q。
u.reset(nullptr)         //将u置为空，令u指向nullptr
```

### 赋值和拷贝

我们可以拷贝或赋值一个将要销毁的unique_ptr

例如： `return unique_ptr<int><new int(p)>`



### 赋值和拷贝

**自定义销毁操作**

```c++
void end_connection(connection *p) {disconnect(*p);}   //定义自己的释放

void f(destination &d){
  connection c = connect(&d);
  unique_ptr<connection,decltype(end_connection)*> p(&c, end_connection);   //当f退出时（即使是由于异常而退出），connection会被正确关闭
}
```



## weak_ptr类

weak_ptr是一种不控制所指向对象生存周期的智能指针，它指向一个由shared_ptr管理的对象。当最后一个指向shared_ptr被销毁时，对象就会被释放。

**weak_ptr支持的常见操作**

```c++
weak_ptr<T> w;           //空weak_ptr
weak_ptr<T> w(sp);       //与shared_ptr指向相同对象的weak_ptr
w = p                    //p可以是一个shared_ptr或一个weak_ptr
w.reset()                //将w置为空
w.use_count()            //与w共享对象的shared_ptr的数量
w.expired()              //如果w.use_count()为0，返回true，否则返回false
w.lock()                 //如果expired为true，返回一个空shared_ptr；否则返回一个指向w的对象的shared_ptr
  
```



## 直接管理内存

### 分配

用new运算符分配内存，使用delete释放new分配的内存

int *p = new int;           //p指向一个动态分配的，未初始化的无名对象

string *ps = new string   //ps指向一个动态分配的空string



**直接初始化**

```c++
int *pi = new int(1024);

string *ps = new string(10,'a');
```



**使用列表初始化**

```c++
vector<int> *pv = new vector<int>{0,1,2,3,4,5,6,7,8};
```



**进行值初始化**：在类型名后加一对空括号即可

```c++
string *ps = new string();      //初始为空string

string *pi = new int();             //初始化为0
```



**动态分配的const对象**

//初始化为一个const int

```c++
const int *p = new const int(1024);
```

### 释放内存

如果分配了内存没有释放，会造成内存空间耗尽，也称为内存泄漏。当内存耗尽时，就无法再重新分配内存。默认情况下当new无法再分配动态内存时，会抛出一个bad_alloc异常。我们可以添加nothrow来阻止它抛出异常，例如：

```c++
int *p1 = new int;  //如果分配失败，new抛出std::bad_alloc
int *p2 = new(nothrow) int;
```



**释放格式：**

delete p;



**技巧：delete之后重置指针**

int *p(new int(43));     //p指向动态内存

auto q = p;                   //q和p指向相同的内存

delete p;                      //p和q均变为无效

p = nullptr;                   //指出p不再绑定到任何对象



## 动态数组

**分配形式**：

int *p = new int[get_size()];    //p指向第一个int



**类型别名**：

```c++
typedef int Array[42];       //Array表示42个int的数组类型
int *p = new Array;          //等价于int *p = new int[42];

```



**初始化**

```c++
int *p = new int[10]();    //10个值初始化为0
string *p2 = new string[10]()    //10个空string
```



**释放**

```c++
delete []p;
```



## 智能指针和动态数组

### 用unique_ptr管理动态数组

```c++
unique_ptr<int[]> up(new int[10]);
up.release();   //自动delete []销毁其指针
```



**支持的操作**

``` c++
unique_ptr<T[]> u(p);       //u指向p指向的动态数组
u[i]                        //返回数组位置i处的对象
```



### shared_ptr管理动态数组

为了使用shared_ptr，必须提供一个删除器

```
shared_ptr<int> sp(new int[10], [](int *p){delet[] p})
sp.reset();   //使用delete[] p;
```



**局限：未定义下标运算符，且不支持指针的算术运算**

```c++
for(size_t i = 0; i!=10; i++){
  *(sp.get() +i) = i; //使用get获取动态数组的第一个元素的指针
}
```



## allocator类

使用allocator可以先分配空间，后构造对象。

```c++
allocate<string> alloc;               //可以分配string的allocator对象
auto const p = alloc.allocate(n);     //分配n个未初始化的string
```

**常见操作**

```c++
allocator<T>a;         //可以分配T类型的allocator对象
a.allocate(n);         //分配一段原始的，未构造的内存，保存n个类型为n的对象
a.deallocate(p,n)      //释放从T*指针p中地址开始的内存，n必须为p创建时的大小。前执行a.deallocate之前，必须对这块内存的每个对象调用destroy
a.construct(p, args)    //p必须为类型为T*的指针，指向一块原始内存：args被传递给类型为T的构造函数
a.destroy()             //p为T*类型的指针，此算法对p指向的对象执行析构函数
  
```



```
auto q = p;
alloc.construct(q++,10,'c');
alloc.construct(q++,"hi");
```



**警告：**

不能在未构造对象的情况下使用原始内存



**销毁动态数组**

```c++
while(q != p){
  alloc.destroy(--q);    //释放真正构造的string
}
```



拷贝和填充未初始化内存的算法：

```
//这些函数在给定目的位置创建元素，而不是由系统分配内存给它们
uninitialized_copy(b,e,b2);     //从迭代器b和e指出的输入范围中拷贝元素到迭代器b2指定的未构造的原始内存中。b2指向的内存必须足够大，能容纳输入序列中元素的拷贝
uninitialized_copy_n(b,n,b2) 从迭代器b指向的元素开始，拷贝n个元素到b2开始的内存中
uninitialized_fill(b,e,t)    在迭代器b和e指定的原始内存中创建对象，对象的值均为t的拷贝
uninitialized_fill_n(b,n,t)  从迭代器b指向的内存地址开始创建n个对象。b必须指向足够大的未构造的原始内存，能容纳给定数量的对象

```

```c++
//假设有一个int的vector vi，希望将其内容拷贝到动态内存中
//分配比vi中元素所占用空间大一倍的动态内存
auto p = alloc.allocate(vi.size()*2);

//通过拷贝vi中的元素来构造从p开始的元素
auto q = uninitialized_copy(vi.begin(), bi.end(), p);

//将剩余元素初始化为42
uninitialized_fill_n(q,vi.size(),42);
```

