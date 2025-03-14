---
date: 2025-02-18
draft: true
categories:
  - C++
tags:
  - C++
authors:
  - admin
---

# *C++ Concurrency in action* 阅读记录

《C++并发编程实战》阅读记录

<!-- more -->

## C++线程基础

### 线程发起

`std::thread thread_name(thread_func, arg1);`

### 线程等待

主线程等待子线程退出
`t1.join();`

### 使用仿函数作为参数

`std::thread t1{ task() };`

### lambda表达式



### 线程detach

线程分离为守护线程
``