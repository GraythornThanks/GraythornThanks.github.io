---
date: 2024-12-25
categories: 
  - C++
tags:
  - C++
  - Python
  - CMake
authors:
  - admin
---

# C++ 和 Python 混合编程

本文介绍了如何在 C++ 与 Python 的互相调用和使用 Python 辅助 CMake 构建 C++ 项目，文中默认读者使用CMake构建C++代码

<!-- more -->

## C++ 中使用 Python

为什么要引入python：

- 快速开发：在游戏引擎中，通常使用 C++ 编写性能敏感的核心功能，但在修改代码时，任何修改都会导致长时间的重新编译，于是在性能不敏感的地方通常会引入lua、python等解释性语言完成
- 热更新：C++作为编译性语言，在完成编译后的代码在运行时无法修改，解释性语言可以在运行时动态修改代码，从而可以实现热更新
- 辅助构建：在CMake中，可以使用Python脚本替代一些CMake配置

想要在 C++ 代码中运行Python代码，需要以下条件：

- Python解释器
- Python头文件Python.h的可用性
- Python运行时库libpython

### CMake 中配置 Python

`find_package()` 常用于寻找C++库，也可以用来找到 Python 解释器

在比较新的CMake版本中（3.12版本后）：<br>
```cmake
find_package(Python COMPONENTS Interpreter Development REQUIRED)
```
将会找到Python解释器和库路径，并设置一些CMake变量

- `PYTHONINTERP_FOUND` 是否找到 Python 解释器
- `PYTHON_EXECUTABLE` Python 解释器可执行文件的完整路径
- `PYTHON_VERSION_STRING` Python 解释器的完整版本信息
- `PYTHON_INCLUDE_DIRS` Python 头文件的目录
- `PYTHON_LIBRARIES` Python 库文件的目录

找到后将可执行文件链接Python库<br>
```cmake
target_include_directories(YourTarget PRIVATE ${PYTHON_INCLUDE_DIRS})
target_link_libraries(YourTarget PRIVATE ${PYTHON_LIBRARIES})
```

在比较旧的CMake版本中（3.12版本前）：<br>

如何查找 Python 解释器？<br>
```cmake
find_package(PythonInterp)
```
当找到 Python 解释器后，并会附带设置一些CMake变量

如何查找Python库的路径？<br>
```cmake
find_package(PythonLibs ${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR} EXACT REQUIRED)
```

> ❓如何指定寻找特定的 Python 解释器版本<br>
> 📢`find_package(PythonInterp 3.10)`

当找到 Python 解释器后，即可在CMake中执行python代码并捕获输出
```cmake
execute_process(
 COMMAND
 ${PYTHON_EXECUTABLE} "-c" "print('Hello, world!')"
 RESULT_VARIABLE _status
 OUTPUT_VARIABLE _hello_world
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE
  )
```

> ❓Python解释器未安装在标准目录，find_package()未找到<br>
> 📢使用`-D`参数告知cmake Python解释器路径<br>
> &emsp; `cmake -D PYTHON_EXECUTABLE=/path/to/python`<br>
> &emsp; 也可以以同样的方式告知Python头文件和库文件路径<br>
> &emsp; `PYTHON_INCLUDE_DIRS`和`PYTHON_LIBRARIES`

### 将Python解释器嵌入到C++代码中

Python 实际上就是一个C库，在Python的安装文件夹中你将能找到 `python{版本号}.dll`<br>
当你完成了对Python库的链接，就可以直接在C++代码中调用Python

```cpp
#include <Python.h>

int main(int argc, char *argv[])
{
  Py_SetProgramName(argv[0]);
  Py_Initialize();
  PyRun_SimpleString("print 'Hello Python!'\n");
  Py_Finalize();
  return 0;
}
```

该程序将直接输出<br>
`Hello Python!`

### 在C++中使用Python模块

Todo

## Python 中调用 C/C++

1. 直接通过ctypes库调用C/C++库
2. 使用C/C++为Python编写扩展模块


Todo



