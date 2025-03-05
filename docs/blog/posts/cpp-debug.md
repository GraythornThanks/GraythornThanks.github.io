---
date: 2025-1-13
draft: true
categories:
  - C++
tags:
  - C++
  - Debug
authors:
  - admin
---

# C++ Debug

Todo

<!-- more -->

## 引发异常的常见问题

1. 变量未初始化
2. 死循环
3. 内存越界
4. 内存泄漏
5. 空指针与野指针
6. 内存访问违例
7. 栈内存被当成堆内存去释放
8. 线程栈溢出
9. 函数调用约定不一致导致栈不平衡
10. 库与库之间不匹配
11. 死锁
12. GDI对象接近或达到1万个导致异常
13. 对包含C++类成员的结构体进行memset操作
14. 模块注入到程序中导致程序出现异常
15. 添加日志打印覆盖了lasterror的值
16. 其他

## 常见思路

使用IDE调试
- Debug和Release下的调试
- VS附加到进程调试
- Windbg附加到进程调试

添加日志打印
- 通过日志查看代码执行轨迹和业务走向
- 打印关键变量值
- 分块注释代码
通过注释排除问题代码范围
- 定位具体问题代码段
数据断点
- 监控内存数据变化
- 定位变量值异常修改位置
历史版本比对法
- 对比不同版本查找问题引入时间点
缩小代码排查范围
- Windbg分析
- 静态分析dump文件
- 动态调试目标进程
- IDA反汇编分析
- 查看汇编代码
- 分析底层执行细节
辅助分析工具
- SPY++
- Dependency Walker
- Process Explorer
- GDIView等工具

## Windows

### 工具

#### Visual Studio

##### Debug和Release下的差异

###### 未初始化内存

对于Debug模式下未初始化内存`msvc`设置的特殊值

- 0xcccccccc
- 0xcdcdcdcd
- 0xfeeefeee
- 0xdddddddd
- 0xabababab
- 0xababcafe
- 0xbaadf00d
- 0xbadcab1e
- 0xbeefcace

##### 在Debug下遇到报错弹窗，点击重试，查看函数调用堆栈

##### 调试时程序与调试器都发生了闪退，可以尝试到VS的Output窗口查看错误信息

##### 调用OutputDebugString接口将打印日志输出到调试器输出窗口中

##### visual studio 中自带的异常中断选项

##### 条件断点与临时过滤条件

##### 数据断点

##### 附加到目标上进行调试

##### 汇编

#### Spy++

#### 库依赖关系 Dependency Walker

#### Clipbrd

#### GDIView

#### Process Explorer

#### Process Monitor

#### API Monitor

#### Windbg



#### IDA

#### Wireshark

## Linux



