---
title: C++ 智能指针需要注意的点
date: 2021-10-24 11:10:21
updated:
categories: [CPP]
tags: [CPP,指针指针,weak_ptr]
---
C++11提供了三种智能指针：std::shared_ptr, std::unique_ptr, std::weak_ptr，使用时需添加头`<memory>`。

## 智能指针与普通指针混用
<!-- more -->
不能将一个原始指针直接赋值给智能指针。

错误示例：

```c++
int *p = &num;
std::shared_ptr<int> ptr = p; // Error 当 ptr 引用计数变为 0 时，p 将变成空悬指针
```

正确示例：尽量不要将 普通指针和智能指针`混用`

```c++
std::shared_ptr<int> ptr = std::make_shared<int>(123);  
std::shared_ptr<int> ptr(new int(123)); 
```

std::make_shared 返回shared_ptr 智能指针，减少了一次内存分配操作，性能更好
std::make_shared 性能测试 http://tech-foo.blogspot.com/2012/04/experimenting-with-c-stdmakeshared.html



## 循环引用

循环引用将无法正常释放资源，应尽量避免。

```c++
#include <iostream>
#include <memory>

class Node{
public:
    std::shared_ptr<Node> next;
    Node()
    {
        std::cout<<"Node construct"<<std::endl;
    }
    ~Node(){
        std::cout << "Node destroy" << std::endl;
    }
};

int main(){
    std::shared_ptr<Node> A = std::make_shared<Node>();  // A 的引用计数为 1
    std::shared_ptr<Node> B = std::make_shared<Node>();  // B 的引用计数为 1

    A->next = B;   // B 的引用计数为 2
    B->next = A;   // A 的引用计数为 2
    // 离开作用域后，A 的引用计数减 1，B 的引用计数减 1。都不为 0，A 和 B 都没有自动释放
    
    return 0;
}
```

运行结果

```shell
Node construct
Node construct
```



## weak_ptr 不可直接使用对象

要使用 weak_ptr 指向的对象，需要将 weak_ptr 提升为 shared_ptr

```c++
#include <iostream>
#include <memory>

class Node{
public:
    std::shared_ptr<Node> next;
    Node()
    {
        std::cout<<"Node construct"<<std::endl;
    }
    ~Node(){
        std::cout << "Node destroy" << std::endl;
    }
};

int main(){
    std::shared_ptr<Node> A = std::make_shared<Node>();  // A 的引用计数为 1
    std::weak_ptr<Node> wptr = A;        // 定义 weak_ptr 不影响 A 的引用计数
    
    std::shared_ptr<Node> ptr= wptr.lock();   // 将 wptr 提升为 shared_ptr,A 的引用计数为 2
    std::cout<<ptr.use_count()<<std::endl;    // 输出 2

    return 0;
}
```