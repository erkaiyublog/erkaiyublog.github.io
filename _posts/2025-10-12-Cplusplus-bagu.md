---
published: false
title: 对一些C++八股题的摘录
tags: CN C++
---

由于最近在秋招，笔试和面试中都遇到了大量的C++八股题，因此也不断暴露自己对C++的一无所知。在此整理一些遇到过的、网上见到的、以及自己延伸的C++细节问题，以供日后回顾。

***目录***
* TOC
{:toc}

# 内存分配
常见的内存分配是在栈或者堆上。注意一个程序运行时被分配的栈大小一般是8MB（这是由操作系统、编译器和运行时的设置决定的）。

内存分配的问题一般是围绕堆上的内存，因为栈上的内存有自动分配和释放的特点。

分配和释放内存常用的是```malloc/free```和```new/delete```两组关键字，前者继承自C语言。

一些我偶尔会忘记的细节：
```cpp
// 1. delete[]的使用
int* array = new int[100]; // array是整数指针，但堆上分配了100个整数的内存
delete[] array; // delete带上[]以表明释放的是整个数组的内存，此处用delete会UB

// 2. new与malloc失败的处理
try { // new的失败会触发exception，所以应该用try/catch来处理
    int* p = new int[1000000000000];
} catch (const std::bad_alloc& e) {
    std::cerr << "Allocation failed: " << e.what() << '\n';
}

int* p = (int*)malloc(sizeof(int) * 1000000000000);
if (!p) { // malloc的失败只会返回nullptr
    std::cerr << "malloc failed\n";
}
```

## ```new```与```malloc```有什么区别
这个应该算较为经典的八股题。主要区别如下：
1. ```new```会调用构造函数，```malloc```不会。
2. ```malloc```失败时会返回```nullptr```，```new```失败时会产生```exception```。
3. ```malloc```返回的是```void*```，需要显式类型转换。
4. 传参不同，```malloc```需要计算字节大小，而```new[]```只需要提供数量。

## C++程序的内存布局
这个除了栈和堆之外应该需要清楚其他内存区域的存在以及功能。通常的模型是栈在高地址，从高向低增长。
1. 堆：由```new/malloc```分配的内存。生命周期由程序员控制，空间大但分配速度慢，容易产生内存泄漏。
2. 栈：存放局部变量、函数参数等。由编译器自动管理，效率高，但空间有限。函数结束自动释放。
3. 全局/静态存储区：存放全局变量、静态变量（包括```static```局部变量）。在程序开始时分配，结束时释放。其中包含一个```data```段（已初始化的全局/静态变量）和一个```bss```段（未初始化的全局/静态变量）。
4. 常量存储区：存放字符串常量和其他常量。通常只读。
5. 代码区：存放程序的二进制代码（函数体）。

## 内存泄漏，悬空指针，野指针和双重释放
主要是定义需要记住：
1. 内存泄漏：程序中已动态分配的堆内存由于某种原因未能被释放，导致系统内存的浪费，最终可能耗尽系统内存。经典的就是调用了```new```没有调用```delete```。
2. 悬空指针：指针指向的内存已被释放，但指针本身仍然存在。例如```int* p = new int(10); delete p;```
3. 野指针：未初始化的指针。它指向随机的内存地址。
4. 双重释放：对同一块堆内存进行了两次或多次```delete```操作，这会导致UB。

# RAII与智能指针
RAII全称是Resource Acquisition Is Initialization，“资源获取即初始化”。核心思想是在构造函数中完成对资源的获取，在析构函数中完成对资源的释放，这样就不用显示地调用```new/delete```。

一个经典的RAII例子：
```cpp
#include <fstream> 
#include <string>

void someFunction() {
    std::ifstream file("data.txt"); // 获取资源，在构造函数中打开文件

    std::string line;
    while (std::getline(file, line)) {
        // ... 处理每一行 ...
    }

    // 释放资源，不需要手动调用file.close()!
} // 当file对象离开作用域时，它的析构函数会自动被调用，析构函数内部会关闭文件。
```

智能指针主要在```<memory>```头文件中。智能指针有以下几种：
1. ```unique_ptr```：独占所有权，同一时间只能有一个指针指向一个对象(可以是指向一个变量的指针也可以是指向一个```new[]```动态分配的数组)。禁止拷贝，允许移动。
2. ```shared_ptr```：共享所有权。通过引用计数来管理生命周期。当最后一个```shared_ptr```被销毁时，对象才会被释放。允许拷贝。
3. ```weak_ptr```：对```shared_ptr```管理对象的弱引用。用于解决```shared_ptr```的循环引用问题。不增加引用计数，不拥有对象的所有权。需要通过```lock()```方法获取一个临时的```shared_ptr```来访问对象。

一些使用智能指针的代码片段：
```cpp
std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>(42);
ptr1->print();

std::unique_ptr<MyClass> ptr2 = std::move(ptr1); // 转移所有权，只能移动
ptr2->print();

std::shared_ptr<Resource> ptr1 = std::make_shared<Resource>();
{
    // 共享所有权
    std::shared_ptr<Resource> ptr2 = ptr1;
    ptr2->use();
}
ptr1->use();

class Node {
public:
    std::string name;
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;  // 使用 weak_ptr 避免循环引用
    
    Node(const std::string& n) : name(n) {
        std::cout << "Node " << name << " created\n";
    }
    
    ~Node() {
        std::cout << "Node " << name << " destroyed\n";
    }
};

auto node1 = std::make_shared<Node>("A");
auto node2 = std::make_shared<Node>("B");

// 建立双向链接
node1->next = node2;
node2->prev = node1;  // weak_ptr 不会增加引用计数

// 访问 weak_ptr需要使用lock()
if (auto shared_prev = node2->prev.lock()) {
    std::cout << "Previous node: " << shared_prev->name << std::endl;
}
```

# 类与结构
类(class)和结构(struct)的区别仅在于默认访问权限。类和结构在语言特性支持上是完全一致的，包括模板、虚函数、继承等所有面向对象特性。二者在编译时被视作相同的。

## 虚函数
```virtual```关键字声明，用于实现运行时的多态。其底层原理是编译器会对每个带有虚函数的类构造```vtable```虚函数表，每个该类的对象都会包含一个隐藏的指针```vptr```，指向其所属类的虚函数表。

派生类重写虚函数时函数的签名应该与基类的虚函数一致，并在定义处加```override```关键字。

## 纯虚函数和抽象类
基类不写任何虚函数的实现，以强制派生类实现自已的函数。包含至少一个虚函数的类就成为了抽象类，抽象类不能创建对象。

## 模版类


## 虚继承

# 锁与线程
# 移动语义
# 编译过程
# Perf性能分析
# 网络栈的实现
# 资源
## 书
* [A Tour of C++](https://www.stroustrup.com/Tour.html): Bjarne Stroustrup写的书，仅256页，适合空闲时翻阅
## Github上的文库
* [cpp_backend_awsome_blog](https://github.com/0voice/cpp_backend_awsome_blog): 有1000篇c++后端相关博文，2023年整理的，这个库的拥有者[@0voice](https://github.com/0voice)还有大量的相关的库
* [Modern-CPP-Programming](https://github.com/federico-busato/Modern-CPP-Programming): ppt形式的C++教材
## 文档
* [cppreference.com](https://cppreference.com/): 看着目录复习就行
