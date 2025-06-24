---
date: 2025-06-11
draft: true
categories:
  - C++
tags:
  - C++
authors:
  - admin
---

# C++ 生命周期与智能指针

<!-- more -->

## 三大存储周期

参考： [cppref](https://cppreference.cn/w/cpp/language/storage_duration)

### 1. 自动存储周期

这种变量在进入其作用域时被创建，在离开其作用域时被销毁。

```cpp
void func() {
    int a = 1; // a 是自动存储周期
    // 直接定义在{}内，俗称"栈上"或"局部变量"
}
```

### 2. 动态存储周期

这种变量在运行时通过new创建，在运行时通过delete销毁。

```cpp
void func() {
    Class *p = new Class;  // *p 是动态存储周期
    delete p;              // 释放动态分配的内存
}
```

### 3. 静态存储周期

这种变量在程序的整个生命周期中都存在。

```cpp
namespace ns { // a b c 是静态存储周期
  class a;
  static class b;
  inline class c;
} // 构造时机：程序启动时/DLL首次加载时；析构时机：程序结束时

int a = 1; // a 是静态存储周期，可以认为全局名字空间是一个特殊的名字空间

struct Other {
    static Class s;
};
Class Other::s; // 定义在类内的静态成员 s 是静态存储周期
```

### 总结

- 自动存储周期：栈上局部变量，自动析构
- 动态存储周期：通过new创建的堆上变量，需要手动delete析构
- 静态存储周期：全局名字空间，程序结束时析构

**动态存储周期的变量需要我们自己管理！**

!!! question "如何更好地管理动态存储周期的变量？"
    RAII: Resource Acquisition Is Initialization <br>
    通过类似栈上的变量自动析构的方式来管理动态存储周期的变量

!!! question "如果直接用delete手动管理？"
    无法保证delete的时机与顺序<br>
    一旦出现多个分支离开作用域，就无法保证delete的顺序

## RAII

RAII (Resource Acquisition Is Initialization) 即：资源获取即初始化

> “资源管理应当与对象的生命周期绑定。这是 C++ 区别于 C 的关键特性之一。” <br>
> —— Bjarne Stroustrup, 《The C++ Programming Language》

**核心思想**

1. 资源绑定对象

    - 将资源（内存、文件句柄、网络连接、锁等）的生命周期与对象的生命周期绑定。
    - 资源在构造函数中获取，在析构函数中释放。

2. 自动管理

    - 当对象离开作用域时（如函数结束、代码块退出），析构函数自动调用，确保资源释放。
    - 即使发生异常，栈回退（stack unwinding）机制也能保证析构函数被执行，避免资源泄漏。

### 实现 RAII 类

TODO

