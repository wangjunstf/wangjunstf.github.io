---
title: C++单例设计模式的两种实现
date: 2021-10-13 23:27:39
updated:
categories:
tags:
---
# 单例设计模式的两种实现
## 单例模式的定义

保证一个类仅有一个实例，并提供一个它的全局访问点，该实例被所有程序模块所共享。

那么就必须保证：

* 该类不能被实例化
* 该类不能被复制。

对于 C++，意味着：它的构造函数，拷贝构造函数和拷贝赋值运算符不能被公开调用。
<!-- more -->


单例模式通常有两种实现模式：

* 懒汉式单例
* 饿汉式单例



## 懒汉式单例

单例实例在第一次被使用时才进行初始化，这叫做延迟初始化。

### 非线程安全实现

**静态指针 + 用到时初始化**

版本 1：

```c++
class Singleton
{
public:
static Singleton& getInstance()
{
    if (!value_)
    {
        value_ = new Singleton();
    }
    return *value_;
}
private:
    Singleton();
    ~Singleton();
  	Singleton(const Singleton&);
		Singleton& operator=(const Singleton&);
    static Singleton* value_;
};

// 初始化成员变量
Singleton* Singleton::value_ = nullptr;
```

在单线程下可以正确，但在多线程下就可能出现问题。比如当A线程调用getInstance()，并进入 if 作用域内，在调用 new 之前，时间片用完了，就进入就绪队列，此时线程B也调用getInstance()，并成功执行 new 分配了对象。随后运行线程A，线程A 也调用了new 分配了对象。就不满足单例模式仅有一个实例的要求。

还有一个问题，可能造成内存泄漏，因为分配的动态内存没有主动释放。

针对第二个问题，可以改为以下版本：

版本3:

```c++
class Singleton
{
public:
    static Singleton* getInstance()
    {
        if (!value_)
        {
            value_ = new Singleton();
        }
        return value_;
    }

private:
    class Deletor
    {
    public:
        ~Deletor()
        {
            if (Singleton::getInstance != nullptr)
                delete Singleton::value_;
        }
    };
    static Deletor deletor;
    Singleton();
    ~Singleton();
    Singleton(const Singleton &);
    Singleton &operator=(const Singleton &);
    static Singleton* value_;
};

// 初始化成员变量
Singleton *Singleton::value_ = nullptr;
```

版本 3可以解决 内存泄漏的问题。在类内定义了一个私有的 static 全局变量，因为程序在结束时析构全局变量。

### 线程安全实现

还有多线程不安全问题：

方法1: 给临界区加锁

```c++
static Singleton* getInstance()
    {
        lock_guard<mutex> locker(mtx_);
        if (!value_)
        {
            value_ = new Singleton();
        }
        return value_;
    }
```

程序只是在初始化的时候需要加锁，初始化完之后就不需要锁了。使用双检测锁可以解决这个问题：

```c++
static Singleton* getInstance()
    {
        if(!value_){
            lock_guard<mutex> locker(mtx_);
            if (!value_)
            {
                value_ = new Singleton();
            }
        }
        return value_;
    }
```

此时看似已经解决问题了，但还有一个隐含的问题：memory model。在某些内存模型中，或者由于编译器优化，或者运行时优化等原因，调用 getInstance()时虽然，value_ 不为 nullptr,但是还没有完全构造，此时调用的对象就是不完整的。出现问题的原因就是：`if(!value_)` 与 `value_ = new Singleton();`没有正确同步，在某种情况下，new 返回了地址给value_，但Singleton 还没有完全构造，当另一个线程调用时将不会进入if从而返回了不完全的实例对象给用户使用，造成了严重的错误。C++11引进了memory model，提供了Atomic实现内存的同步访问，即不同线程总是获取对象修改前或修改后的值，无法在对象修改期间获得该对象。

```c++
static atomic<Singleton *> value_;   
```

将 value_ 定义为atomic<Singleton *>，此时它便具有原子属性。

完整代码如下

```c++
#include <mutex>
#include <iostream>
#include <atomic>
class Singleton
{
public:
    static Singleton* getInstance()
    {
        if(!value_){
            lock_guard<mutex> locker(mtx_);
            if (!value_)
            {
                value_ = new Singleton();
            }
        }
        return value_;
    }

private:
    class Deletor
    {
    public:
        ~Deletor()
        {
            if (Singleton::getInstance != nullptr)
                delete Singleton::value_;
        }
    };
    static Deletor deletor;
    Singleton();
    ~Singleton();
    Singleton(const Singleton &);
    Singleton &operator=(const Singleton &);
    static atomic<Singleton *> value_;                   
    static std::mutex mtx_;
};

// 初始化成员变量
atomic<Singleton *> Singleton::value_{nullptr};
```



### 更优雅的方式

**局部静态变量法**

C++11规定了local static在多线程条件下的初始化行为，要求编译器保证了内部静态变量的线程安全性。在C++11标准下，《Effective C++》提出了一种更优雅的单例模式实现，使用函数内的局部静态变量。只有当第一次访问`getInstance()`方法时才创建实例。

```c++
class Singleton
{
public:
    static Singleton& getInstance()
    {
        static Singleton value_;
        return value_;
    }

private:
    Singleton();
    ~Singleton();
    Singleton(const Singleton &);
    Singleton &operator=(const Singleton &);
};
```



## 饿汉式单例

饿汉版（Eager Singleton）：指单例实例在程序运行时被立即执行初始化。

**直接定义静态对象**

```c++
class Singleton
{
public:
    static Singleton& getInstance()
    {
        return value_;
    }

private:
    Singleton();
    ~Singleton();
    Singleton(const Singleton &);
    Singleton &operator=(const Singleton &);
    static Singleton value_;
};
```

在 main 函数之前创建，不存在线程安全问题。但是潜在问题在于no-local static对象（函数外的static对象）在不同编译单元中的初始化顺序是未定义的。也即，static Singleton value_;和static Singleton& getInstance()二者的初始化顺序不确定，如果在 value\_ 初始化完成之前调用 getInstance() 方法会返回一个未定义的实例。



**静态指针 + 类外初始化时new空间方式**

```c++
class Singleton
{
protected:
    Singleton() {}

private:
    static Singleton *p;

public:
    static Singleton *initance(){return p;};
};

Singleton *Singleton::p = new Singleton;
```

## 参考

https://zhuanlan.zhihu.com/p/37469260