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

虚函数用```virtual```关键字声明，用于实现运行时的多态。其底层原理是编译器会对每个带有虚函数的类构造```vtable```虚函数表，每个该类的对象都会包含一个隐藏的指针```vptr```，指向其所属类的虚函数表。这里一个经常问的问题是“虚函数表是每个类独有还是每个对象独有？”应该是每个类。

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

模版声明时```template```后面可以是```class```也可以是```typename```，前者比后者更早被引入C++。举个混用的例子：

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
# 左值右值
需要注意的是```lvalue```和```rvalue```的l与r并不直接代表left和right。```lavlue = locator value```，```rvalue = read value```。

一个浅显的理解是左值是可以获得其内存地址值的。

对左值右值的理解需要涉及一些其他的(Value Categories)概念：
1. ```glvalue``` 泛左值：包括传统的左值和将亡值。它们一般有名字，可以确定一个对象或函数。
2. ```prvalue``` 纯右值：就是传统意义上的右值。
3. ```xvalue``` 将亡值：Expring Value, 它是C++11引入的新概念。它代表一个生命周期即将结束、其资源可以被“移动”走的对象。它同时具有左值和右值的部分特性。它是“可以被重用的左值”。通常通过```std::move```转换而来，或者是一个返回右值引用的函数调用。  
4. ```lvalue``` 左值：定位值，有持久状态、有名字、可以取地址的。
5. ```rvalue``` 右值：临时值，一个临时的、短暂的、没有名字、不能被取地址的“值”。C++11之后，右值包括了纯右值和将亡值。

一些例子：
```cpp
// 左值
++i; // 可以写 (++i) = 10; 会将10赋值给i
int x = 5; // x是左值

// 右值
5;
int y = x + 1; // x + 1是右值
```

一个宏观的理解：
```
        expression
          /     \
     glvalue   rvalue
       /  \      /  \
   lvalue   xvalue   prvalue
```

## 左值引用
即传统的C++“引用”。用的是```&```符号。特点是引用的值必须被初始化，且一旦绑定就不能再指向其他对象。需要注意的是```const &```可以绑定右值。这是因为“只读”操作不会破坏右值的临时性。

一些简单的例子：
```cpp
int a = 5;
int &ref_a = a; // 左值引用指向左值，编译通过
int &ref_a = 5; // 左值引用指向了右值，会编译失败!!
const int &ref_a = 5;  // const的引用是可以对右值的，因为const左值引用不会修改指向值
```

## 右值引用
右值引用是C++11引入的，用```&&```表示。指的是一个即将被销毁的对象的别名。它“接管”了该临时对象的资源。主要用来延长临时对象的生命周期，或者“窃取”其资源。只能绑定到右值，不能直接绑定到左值（除非使用```std::move```强制转换）。

右值引用的写法如下：
```cpp
int&& r = 10; // 10是右值
r = 20; // 修改了这个临时值
```

可见右值引用在这种写法下没什么太大意义。它的关键在于可以右值引用指向与```std::move```结合。```std::move```函数会返回一个左值的右值引用（准确的地说，是一个将亡值），即告诉编译器某个左值的资源将被偷走（但窃取的具体是由程序员实现），被move from的左值在这之后的状态是unspecified，它没有立即被摧毁，并且可以在之后被正常摧毁，但不应该再使用它。语法是```auto&& r = std::move(l)```。

一个实际的例子，运用了移动构造函数来高效实现移动，避免深拷贝：
```cpp
class MyString {
    char* m_data;
public:
    // 移动构造函数：参数是右值引用
    MyString(MyString&& other) noexcept {
        std::cout << "Move Constructor Called" << std::endl;
        m_data = other.m_data;  // “窃取”资源
        other.m_data = nullptr; // 将原对象置为空，防止其析构时释放资源
    }

    // ... 其他构造函数、析构函数等 ...
};

int main() {
    MyString s1;
    // MyString s2(s1);    // 这会调用拷贝构造函数（如果存在）
    MyString s2(std::move(s1)); // 使用 std::move 将 s1 强制转换为右值，
                                // 从而调用移动构造函数，高效转移资源。
                                // 此后s1的状态变成unspecified，到作用域解释正常释放资源
    return 0;
}
```

## 移动语义
动机是避免昂贵的拷贝（例如对于```vector```, ```string```对象简单赋值时都是深拷贝实现）。移动语义的核心思想是窃取即将消亡的对象的资源。

移动语义是基于右值引用的，一般是将A的资源窃取给B时，先用```std::move(A)```来得到A的将亡值，然后利用B自身定义的移动构造函数或移动赋值运算符来窃取A。

以下是一个较完整的例子：
```cpp
#include <iostream>
#include <cstring>
#include <utility> // for std::move

class String {
private:
    char* m_data;
    size_t m_size;

public:
    // 1. 普通构造函数
    String(const char* str = "") {
        m_size = strlen(str);
        m_data = new char[m_size + 1];
        strcpy(m_data, str);
    }

    // 2. 拷贝构造函数 (深拷贝)
    String(const String& other) {
        m_size = other.m_size;
        m_data = new char[m_size + 1];
        strcpy(m_data, other.m_data);
    }

    // 3. 移动构造函数 (资源窃取)
    // 注意：参数是非常量右值引用 String&&
    String(String&& other) noexcept {
        // 直接窃取 other 的资源
        m_data = other.m_data;
        m_size = other.m_size;
        // 将 other 置于有效但可析构的状态
        other.m_data = nullptr;
        other.m_size = 0;
    }

    // 4. 拷贝赋值运算符 (深拷贝)
    String& operator=(const String& other) {
        if (this != &other) { // 防止自赋值
            delete[] m_data; // 释放原有资源
            m_size = other.m_size;
            m_data = new char[m_size + 1];
            strcpy(m_data, other.m_data);
        }
        return *this;
    }

    // 5. 移动赋值运算符 (资源窃取)
    String& operator=(String&& other) noexcept {
        if (this != &other) { // 防止自赋值（虽然不常见，但安全起见）
            delete[] m_data;   // 释放自己的原有资源
            // 窃取 other 的资源
            m_data = other.m_data;
            m_size = other.m_size;
            // 将 other 置于有效但可析构的状态
            other.m_data = nullptr;
            other.m_size = 0;
        }
        return *this;
    }

    // 析构函数
    ~String() {
        delete[] m_data; // delete nullptr 是安全的!
    }
};

int main() {
    String str1("Hello");

    // 使用拷贝构造创建 str2
    String str2(str1); // 调用拷贝构造函数

    // 使用移动构造创建 str3
    // String("World") 是一个临时对象（右值），所以会调用移动构造函数
    String str3(String("World"));

    // 使用 std::move 强制移动 str1 到 str4 
    // std::move(str1) 将左值 str1 转换为右值引用，允许“移动”
    // 此后，str1 不再拥有有效的字符串数据！
    String str4(std::move(str1));
    std::cout << "str4: ";

    // 移动赋值操作 
    String str5;
    // 将 str4 的资源移动给 str5
    str5 = std::move(str4);

    return 0;
}
```

## 完美转发
完美转发指的是在编写泛型函数（尤其是模板函数）时，将一个或多个参数连同其原始类型（值类型、左值引用、右值引用）以及常量性，毫无保留地、原封不动地传递给另一个函数。

首先需要理解一个叫引用折叠的概念，在模板推导的语境下，引用的引用会被折叠：
1. ```T& &```, ```T& &&```, ```T&& &```都会折叠成```T&```。
2. ```T&& &&```会折叠成```T&&```。

根据这个折叠的性质，如果直接写两个模版函数，一个对左值，一个对右值，且假设这中间还有一层函数用于调用这两个函数中的一个，则如果简单地对传入参数进行调用，由于引用折叠，一定会调用左值的实现，如果强行用```std::move```则一定会调用右值实现。于是需要```std::forward```来完美转发原本的左右值。以下为一个例子：

```cpp
template<typename T>
void print(T & t){
    std::cout << "Lvalue ref" << std::endl;
}

template<typename T>
void print(T && t){
    std::cout << "Rvalue ref" << std::endl;
}

template<typename T>
void testForward(T && v){ 
    print(v);   //v此时已经是个左值了,永远调用左值版本的print
    print(std::forward<T>(v)); //保留左右值的性质
    print(std::move(v)); //永远调用右值版本的print

    std::cout << "======================" << std::endl;
}

int main(int argc, char * argv[])
{
    int x = 1;
    testForward(x); //实参为左值
    testForward(std::move(x)); //实参为右值
}
```

## 引用和指针的区别
引用本质上是变量的别名，一旦初始化后就不能更改绑定且不能为空，使用起来像普通变量一样直接；而指针是一个存储内存地址的变量，可以重新指向不同对象、可以为空。

# 锁
C++的一些常见锁包括：
1. ```mutex```，只能一个线程独占，其他的线程用```lock()```访问会被block。但也可以用```try_lock()```来尝试访问，失败时不会block。
2. ```shared_mutex```，允许多个线程同时读取，但写入时独占。
3. ```recursive_mutex```，允许同一线程多次锁定，解决了递归函数中的死锁问题（一个线程在递归函数哪多次取同一个锁，普通的互斥锁会在第二次时死锁）。
4. ```timed_mutex```，允许尝试在指定时间内获取锁，避免无限期的block。
5. ```lock_guard```，RAII互斥锁包装器，实现构造时自动加锁，析构时自动解锁，确保异常退出时锁的释放。
6. ```unique_lock```，更灵活的RAII互斥锁包装器，支持延迟锁定，手动解锁和锁的所有权转移，比```lock_guard```更灵活。

# 线程池
面试中可能会出现类似“简述线程池的编写方式”的题目（我曾被问过）。这里记一下线程池的大概思路。

线程池的核心组件包括：
1. 任务队列：用```queue```或者```priority_queue```来维护，用```mutex```保护。
2. 工作线程集合：用```vector```存储，记录所有创建的工作线程。
3. 同步机制：```mutex```用于保护任务队列，```condition_variable```表示线程等待或唤醒的状态，```atomic<bool>```作为停止标志。

工作流程：
1. 初始化阶段，创建固定数量的工作线程，每个线程循环执行：等待任务->取任务->执行任务。
2. 任务提交是由用户提交到队列，线程池会唤醒一个等待的线程去执行任务。
3. 当停止时，设置停止标志，唤醒所有线程并等待所有线程结束。

# 资源
## 书
* [A Tour of C++](https://www.stroustrup.com/Tour.html): Bjarne Stroustrup写的书，仅256页，适合空闲时翻阅

## Github上的文库
* [cpp_backend_awsome_blog](https://github.com/0voice/cpp_backend_awsome_blog): 有1000篇c++后端相关博文，2023年整理的，这个库的拥有者[@0voice](https://github.com/0voice)还有大量的相关的库
* [Modern-CPP-Programming](https://github.com/federico-busato/Modern-CPP-Programming): ppt形式的C++教材

## 文档
* [cppreference.com](https://cppreference.com/): 看着目录复习就行

## 视频
* [CppCon 2018: Jonathan Boccara “105 STL Algorithms in Less Than an Hour”](https://www.youtube.com/watch?v=2olsGf6JIkU): STL里一些算法的实现