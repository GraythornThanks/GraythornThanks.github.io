---
date: 2025-01-02
categories:
  - C++
tags:
  - MSVC
  - UTF-8
  - C++
authors:
  - admin
---

# MSVC拥抱UTF-8

msvc默认使用locale相关的编码设置，这通常是跨平台不友好的，甚至将Visual Studio的工程文件用vscode打开都会导致乱码，文中将介绍如何设置msvc拥抱UTF-8编码

<!-- more -->

## 编码的基础知识

为什么是UTF-8？请参考[UTF-8 遍地开花](https://utf8everywhere.org/zh-cn)

### 字符集

!!! question "什么是字符集？"
    字符集是字符的集合，字符集中的每个字符都有一个唯一的标识符，计算机不能直接存储字符，而是用数字表示字符

常见字符集：

- ASCII 基本拉丁字母 共7位，128个字符
- Latin-1 补充了ASCII字符集，保持原有不变，追加了128到255的映射关系 共8位，256个字符
- Unicode 为世界上每个字符分配一个16位的二进制码，出于兼容考虑，在0到256的映射关系与ASCII、Latin-1一致，Unicode 字符映射的整数范围是 0x0 到 0x10FFFF
- GB2312 Todo
- GBK Todo
- GB18030 Todo

当计算机存储字符时，实际是存储了对应的码点，码点 `code point` 是字符集中的字符的唯一标识符

### 字符编码

!!! question "字符编码是什么？"
    字符集只规定了字符的映射，未规定字符在内存中如何存在，可以认为字符编码是字符集的实现，将字符的整数编号序列化为计算机可直接存储的一个或多个实际存在的整数类型

Unicode 字符可以选用以下这些字符编码来序列化：

- UTF-32: 每个 Unicode 字符用 1 个 uint32_t 整数存储。
- UTF-16: 每个 Unicode 字符用 1 至 2 个 uint16_t 整数存储。
- UTF-8: 每个 Unicode 字符用 1 至 4 个 uint8_t 整数存储。

翻译出来的这些小整数叫 码位 (code unit)。uint8_t就是Utf-8的码位

#### UTF-32

UTF-32一个码点对应一个码位，是一种定长编码

**优点：**

- 可以直接索引、切片、反转，不会破坏字符，方便文本处理

**缺点：**

- 浪费空间

#### UTF-8

Unicode 编码字符时，特意把常用的字符靠前排列了，UTF-32虽然方便文本处理但过于浪费空间，不利于存储

世界上常用语言文字都被刻意编码在了 0 到 0xFFFF 区间内，超过 0x10000 的基本都是不常用的字符

在 0 到 0xFFFF 区间内，同样有按照常用度排序：

- 0 到 0x7F 是（欧美用户）最常用的英文字母、阿拉伯数字、半角标点。
- 0x80 到 0x7FF 是表音文字区，常用的注音字母、拉丁字母、希腊字母、西里尔字母、希伯来字母等
- 0x800 到 0xFFFF 是表意文字，简繁中文、日文、韩文、泰文、马来文、阿拉伯文等
-0x10000 到 0x10FFFF 是不常用的稀有字符，例如甲骨文、埃及象形文字、Emoji 等

UTF-8 把一个码点序列化为一个或多个码位，一个码位用 1 至 4 个 uint8_t 整数表示。

- 0 到 0x7F 范围内的字符，用 1 个字节表示。
- 0x80 到 0x7FF 范围内的字符，用 2 个字节表示。
- 0x800 到 0xFFFF 范围内的字符，用 3 个字节表示。
- 0x10000 到 0x10FFFF 范围内的字符，用 4 个字节表示。

由此可见，UTF-8 作为变长编码，一个码点可能对应多个码位

**优点：**

- 节省空间

**缺点：**

- 数组的长度，不一定是字符串中实际字符的个数。因此，要取出单个字符，需要遍历数组，逐个解析码位。
- 数组的单个元素索引，无法保证取出一个完整的字符。
- 对数组的切片，可能会把一个独立的字符切坏。
- 反转数组，不一定能把字符串的反转，因为可能不慎把一个字符的多个码位反转，导致字符破坏。

#### UTF-16

Todo

#### 字节序问题

多字节的整数类型（如 uint16_t 和 uint32_t）需要被拆成多个字节来存储。拆开后的高位和低位按什么顺序存入内存？

- 大端字节序 (bit endian)：低地址存放整数的高位，高地址存放整数的低位，也就是大数靠前！这样数值的高位和低位和人类的书写习惯一致。

- 小端字节序 (little endian)：低地址存放整数的低位，高地址存放整数的高位，也就是小数靠前！这样数值的高位和低位和计算机电路的计算习惯一致。

> Intel 的 x86 架构和 ARM 公司的 ARM 架构都是小端，C#游戏业务居多为了性能采用小端，而互联网一般规定多字节的数据在网络包中采用大端，Java面向互联网业务居多所以JVM采用大端，而C++通常根据的是机器的大小端

UTF-16 和 UTF-32 的码位都是多字节的，也会有大小端问题

可以进一步将UTF-16和UTF-32的分为：

- UTF-16LE：小端的 UTF-16
- UTF-16BE：大端的 UTF-16
- UTF-32LE：小端的 UTF-32
- UTF-32BE：大端的 UTF-32

如果只在内存的 wchar_t 中使用就不用区分，默认跟随当前机器的大小端。所以 UTF-16 和 UTF-32 通常只会出现在内存中用于快速处理和计算，很少用在存储和通信中

总之，完全基于字节的 UTF-8 是最适合网络通信和硬盘存储的文本编码格式，UTF-16 和 UTF-32 通常只用于内存中，UTF-32 是最适合在内存中处理文本的格式

### BOM

BOM(Byte Order Mark) 端序标记, 是位于码点U+FEFF的统一码字符的名称。

0xFEFF 是一个特殊的不可见字符，一个零宽空格，在不同的编码下会产生不同的结果：

- UTF-8：0xEF 0xBB 0xBF，他会占用 3 字节，而且不会告诉你是大端还是小端，因为 UTF-8 是没有大小端问题的。
- UTF-16：如果是大端，就是 0xFE 0xFF，如果是小端，就是 0xFF 0xFE。
- UTF-32：如果是大端，就是 0x00 0x00 0xFE 0xFF，如果是小端，就是 0xFF 0xFE 0x00 0x00。

所以，端序标记可以用来

1. 区分端序
2. 区分编码

!!! note 
    Windows 环境中，所有的文本文件都被默认假定为 ANSI（GBK）编码，如果你要保存文本文件为 UTF-8 编码，就需要加上 BOM 标志。当 MSVC 读取时，看到开头是 0xEF 0xBB 0xBF，就明白这是一个 UTF-8 编码的文件。这样，MSVC 就能正确地处理中文字符串常量了。如果 MSVC 没看到 BOM，会默认以为是 ANSI（GBK）编码的，从而中文字符串常量会乱码。MSVC 中开启 /utf-8 选项也能让 MSVC 把没有 BOM 的源码文件当作 UTF-8 来解析

### 区域设置

对于中国区 Windows 来说，区域设置默认是 GBK。对于美国区 Windows 来说，区域设置默认是 UTF-8

对于 Linux 用户来说，如果你没有专门修改过，$LC_ALL 默认是 en_US.UTF-8 或 C.UTF-8

!!! note "Windows中的ANSI与Unicode"
    - Windows官方的说法中，ANSI是区域设置中设置的编码格式，ANSI码仅在前128（0-127）个与ASCII码相同，之后的字符全是某个国家语言的所有字符。中国制定了GB2312编码，用来把中文编进去另外，日本把日文编到Shift_JIS里，韩国把韩文编到Euc-kr里，各国有各国的标准。不同语言之间的ANSI码之间不能互相转换，这就会导致在多语言混合的文本中会有乱码
    - Unicode指的是UTF-16
    - UTF-8 指的是 UTF-8 with BOM 而不是正常的 UTF-8

由于在安装系统时通常会选择区域为中国，语言为中文，导致系统默认编码为GBK，以致于msvc会默认使用GBK编码，包括控制台的默认输出也会选择GBK编码


## C++ 中的字符编码

类型|大小|编码|字面量
----------------|-----|----------------|------
Windows`char`   |1字节|取决于区域系统设置|`u8'a'`
Linux  `char`   |1字节|取决于`$LC_ALL`  |`u8'a'`
Windows`wchar_t`|2字节|UTF-16LE        |`L'a'`
Linux  `wchar_t`|2字节|UTF-32          |`L'a'`
C++20 `char8_t`	|1字节|UTF-8            |`u8"hello"`
C++20 `char16_t`|2字节|UTF-16           |`u"hello"`
C++20 `char32_t`|4字节|UTF-32           |`U"hello"`

可见，char和wchar_t都是不跨平台的，而char8_t、char16_t、char32_t等跨平台字符类型从C++20开始才支持

并且根据上述编码相关内容，C++中的字符类型应该为`wchar_t`，字符串类型为`std::wstring`，而`std::string/std::vector<char>`属于字节流

> wchar_t 被提出用于支持宽字符操作，但是从上表中可知该类型是作为标准中的原生类型却不跨平台的！

### 源码字符集和执行字符集

源码字符集(the source character set)：源码文件是使用何种编码保存的

执行字符集(the execution character set)：可执行程序内保存的是何种编码(程序执行时内存中字符串编码)

如何设置源码字符集和执行字符集：

- MSVC 中开启`/utf-8` 或 `/execution-charset:UTF-8` `/source-charset:UTF-8`
- MinGW 中开启`-finput-charset=UTF-8` `-fexec-charset=UTF-8`

### C++ 中跨平台应该怎么做

开发阶段

!!! question "经过上述讨论，导致乱码的主要原因是什么"
    最主要原因：平台api使用的编码与执行字符集不一致，导致平台api调用时乱码

    - 平台api：在控制台输入输出、文件读写、文件路径、标准库调用系统API
    - 执行字符集：程序执行时内存中字符串编码

**一种解决方案是：**

- Windows + MSVC 中开启`/utf-8` 选项
- Windows + MinGW 中开启`-finput-charset=UTF-8` `-fexec-charset=UTF-8`
- Linux + GCC 所有字符集默认为UTF-8
- 所有源码文件统一以 UTF-8 编码保存，且尽量在最前面加上 0xFEFF 这个 BOM 标记，防止 MSVC 当作 GBK 来读取。
- 设置控制台输出编码，设置标准库调用系统API使用的编码

在CMake中这样设置：

```CMake
if (MSVC)
    add_compile_options(/Zc:preprocessor /utf-8 /DNOMINMAX /D_USE_MATH_DEFINES /EHsc /bigobj)
else()
    if (WIN32)
        add_compile_options(-finput-charset=utf-8 -fexec-charset=utf-8)
    endif()
    add_compile_options(-Wall -Wextra -Werror=return-type)
endif()
```

在源代码中：

```C++
int main() {
#if _WIN32
    setlocale(LC_ALL, ".utf-8"); // 设置标准库调用系统 API 所用的编码，用于 fopen，ifstream 等函数
    SetConsoleOutputCP(CP_UTF8); // 设置控制台输出编码，或者写 system("chcp 65001")
    SetConsoleCP(CP_UTF8); // 设置控制台输入编码
#endif
    // 这里开始写你的主程序吧！
    // ...
    return 0;
}
```

IDE 配置中

```.editorconfig
root = true

[*]
charset = utf-8
utf8_bom = true 
```

这样就可以保证，无论你使用什么编译器，无论你使用什么操作系统，无论你使用什么文本编辑器，无论你使用什么编码，你的程序都可以正确的以 UTF-8 编码来读取源码，正确的以 UTF-8 编码来存储字符串常量，正确的把 UTF-8 编码的字符串路径转为 UTF-16 后调用 W 系 API

### Qt 中的编码问题

Todo

### 各种语言/工具选择的内码

| 编码 | 语言/工具 |
| --- | --- |
| ANSI | Lua ext4 openvdb Windows"A"api Python3"byte" HTTP |
| UTF-8 | Rust Go CMake Julia |
| UTF-16 | Windows"W"api Java Qt Word Json NTFS(Windows) |
| UTF-32 | POSIX Kotlin gcc python3"str" |

!!! tip "趋势"
    Java的内码为UTF-16，而Kotlin作为更现代的运行在JVM上的语言，其内码为更方便处理字符串的UTF-32；Rust/Go等常用于实现互联网底层服务的新兴语言，使用了更适合网络传输的UTF-8，显然无论选择拥抱哪种实现，Unicode字符集都是大势所趋

## 各编码的字符串操作

Todo

## 常见乱码

Todo

## 参考资料

- [Unicode and Character Sets](https://learn.microsoft.com/en-us/windows/win32/intl/unicode-and-character-sets)
- [UTF-8 Everywhere](https://utf8everywhere.org/zh-cn)
- [C++ 中避免乱码](https://zhuanlan.zhihu.com/p/627531212)
- [设置locale](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/setlocale-wsetlocale?view=msvc-170#utf-8-support)
- [字符编码那些事](https://142857.red/book/unicode/)