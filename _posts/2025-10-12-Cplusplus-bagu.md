---
published: true
title: 对一些C++八股知识点的复习
tags: CN C++
---

由于最近在秋招，笔试和面试中都遇到了大量的C++八股题，因此也不断暴露自己对C++的一无所知。在此整理一些遇到过的、网上见到的、以及自己延伸的C++细节知识，以供日后回顾。

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

类和结构有些很八股的题，比如我曾被问到过一个空的类对象占内存大小是多少，当时傻乎乎回答0，其实是1字节。

## 虚函数

```virtual```关键字声明，用于实现运行时的多态。其底层原理是编译器会对每个带有虚函数的类构造```vtable```虚函数表，每个该类的对象都会包含一个隐藏的指针```vptr```，指向其所属类的虚函数表。

派生类重写虚函数时函数的签名应该与基类的虚函数一致，并在定义处加```override```关键字。

## 纯虚函数和抽象类
基类不写任何虚函数的实现，以强制派生类实现自已的函数。包含至少一个虚函数的类就成为了抽象类，抽象类不能创建对象。

纯虚函数示例：

```cpp
class Animal {
public:
    virtual void speak() const = 0; // 纯虚函数
};

class Dog : public Animal {
public:
    void speak() const override {
        std::cout << "woof woof!" << std::endl;
    }
};
```

## 虚继承
虚继承是C++中为了解决多重继承时的菱形继承问题（Diamond Problem）而引入的一种机制。

举例：
```cpp
#include <iostream>
using namespace std;

class Base {
public:
    int data;
};

// 普通继承
class Derived1 : public Base { };
class Derived2 : public Base { };

// 多重继承，导致菱形问题
class Diamond : public Derived1, public Derived2 { };

int main() {
    Diamond d;
    d.data = 10; // 运行时错误！不知道是来自 Derived1 还是 Derived2 的 data

    d.Derived1::data = 20; // 需要明确指定路径
    d.Derived2::data = 30; 

    cout << d.Derived1::data << endl; // 输出 20
    cout << d.Derived2::data << endl; // 输出 30

    return 0;
}
```

虚继承通过使用virtual关键字，确保在菱形继承结构中，虚基类（Base）在最终的派生类中只有一个共享的实例。

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    int data;
};

// 使用虚继承
class Derived1 : virtual public Base { }; // 关键字 virtual 在 public 前后都可以
class Derived2 : virtual public Base { };

// 同样的多重继承
class Diamond : public Derived1, public Derived2 { };

int main() {
    Diamond d;
    d.data = 100; // 正确！没有歧义了

    d.Derived1::data = 200;
    d.Derived2::data = 300;

    cout << d.Derived1::data << endl; // 输出 300
    cout << d.Derived2::data << endl; // 输出 300
    cout << d.data << endl;           // 输出 300
    // 它们现在是同一个变量！

    return 0;
}
```

虚继承的实现通常依赖于一个叫做虚基类表指针的机制。当一个类通过虚继承方式派生自一个基类时，编译器会为这个派生类添加一个隐藏的指针成员（通常是vbptr,Virtual Base Table Pointer），这个指针指向一个虚基类表，表中记录了该派生类对象到其虚基类子对象位置的偏移量。

在构造一个最终派生类（如例子中的```Diamond```）的对象时，构造过程会特别处理，首先，直接构造最顶层的虚基类（例子里的```Base```）部分，然后，按照继承顺序构造中间的派生类（例子中的```Derived1```, ```Derived2```）。它们的构造函数会知道如何通过vbptr来定位到已经构造好的、共享的虚基类部分，而不是自己去构造一个新的```Base```。最后，构造最终的派生类自身（例子中的```Diamond```）。

内存对比如下：

```
普通继承:
| Diamond | Derived1::Base | Derived1 | Derived2::Base | Derived2 |
          ^                           ^
          |                           |
          两个独立的 Base 子对象

虚继承:
| Diamond | Derived1 | Derived2 | Shared Base |
                                 ^
                                 |
            Derived1 和 Derived2 共享同一个 Base 子对象
```

题外话：Python对多重继承的处理(MRO, Method Resolution Order)运用的是一种叫C3线性化的算法来强制对类的方法进行先后排序以消除C++中这样的潜在的运行时错误。我在准备八股时偶然从AI口中听说这个Python与C++的不同，于是好奇心驱使下两种都了解了一下，随后我曾问过许多同专业的朋友，都不知道虚继承，谁想到几天后面试某国内量化头部私募，C++八股的第二个问的就是虚继承。。。我其实挺好奇这种虚继承在现实项目中具体有多广泛的运用。

# 模版

```template```后面可以是```class```也可以是```typename```，前者比后者更早被引入C++。举个混用的例子：

```cpp
template<typename T, class U>  
class Example {
private:
    T value1;
    U value2;
    
public:
    Example(T v1, U v2) : value1(v1), value2(v2) {}
    
    T getFirst() { return value1; }
    U getSecond() { return value2; }
    
    void print() {
        std::cout << "Values: " << value1 << ", " << value2 << std::endl;
    }
};
```

## 隐式显式实例化
只需要使用模板，编译器就会在需要的时候，自动生成对应类型的代码，这就是隐式实例化(implicit instantiation)。显式实例化(explicit instantiation)则是告诉编译器“在此处为这个特定的类型生成模板代码”。显式实例化可以显式控制实例化的位置，解决一些多个文件引用同名函数可能存在的冲突。

显式实例化就是一行声明，不会有具体的实现（实现交给编译器），例如：

```cpp
template class MyClass<int>;
template int add<int>(int, int);
```

## 全特化偏特化
特化(specialization)为特定的类型提供一个与通用模板完全不同的实现。应用场景是当需要对一个模版写一个针对某数据类型特殊的实现时。

以下例子展示了函数和类的全特化：

```cpp
template <typename T>
void print(T x) {
    cout << "Generic: " << x << endl;
}

template <>
void print<string>(string x) {
    cout << "String: " << x << endl;
}

template <typename T>
struct Box {
    void show() { cout << "Generic Box\n"; }
};

template <>
struct Box<int> {
    void show() { cout << "Int Box\n"; }
};
```

偏特化(partial specialization)则是仅```template class```有的，它的应用场景是类包含多个模版时，可以特化其中部分的元素并保持其余的为模版。例如：

```cpp
// 类的模版可以偏特化，因为类不能重载
template <typename T1, typename T2>
struct Pair {
    void show() { cout << "Generic Pair\n"; }
};

template <typename T> 
struct Pair<T, int> { // 保留了第一项为模版，而第二项特化成了int
    void show() { cout << "Second is int\n"; }
};

// 函数中没有偏特化，因为函数重载已经实现了该功能！下方为函数重载
template <typename T1, typename T2>
void foo(T1 a, T2 b) { cout << "Generic\n"; }

template <typename T1>
void foo(T1 a, int b) { cout << "T2 is int\n"; }
```
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
