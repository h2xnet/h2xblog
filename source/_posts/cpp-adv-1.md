---
title: C++高级特性系列（1）
date: 2021-06-13 17:25:00
tags: C++
---

# C++新特性系列之向后兼容

## （一）兼容C99

C11之前最新的C标准是1999年制定的C99标准。而第一个C++语言标准却出现在1998年（常被称为C98），之后推出的C++标准也只是进行小的修正，直到出现C++11标准。它们主要有：

（1）预定义宏

\__STDC_HOSTED__  ：如果编译器的目标系统环境中包含完整的标准C库，则此宏值为1，否则为0；

\__STDC__ ：C编译器通过这个宏来表示编译器的实现是否与C标准一致；

\__STDC_VERSION__ ：C编译器通过此宏来表示支持的C标准版本；

\__STDC_ISO_10646__ ：这个宏定义为一个yyyymmL格式的整数常量，例如199712L，用来表示C++编译环境符合某个版本的ISO/IEC 10646标准；

通过上面的这些宏，可以验证机器环境对C和C++库的支持状况。



（2）\__func__预定义标识符

功能：返回所在函数的名字；在轻量调试的时候特别有用，可以快速定位到问题函数；

注意：不允许作为函数的默认返回值；原因是因为在参数声明时，\__func__还未被定义；



(3) _Pragma操作

C/C++标准中，#pragma是一条预处理指令，用它向编译器发送指令进行预处理；

比如#pragma once用于告诉编译器，这个文件只加载一次，功能与以下指令等效：

`#ifndef`

`#define`

`#endif`

再比如下面语句用于静态加载库链接文件: 

`#pragma comment(lib, "user32.lib")` 

而在C++高级特性中，提供了_Pragma操作符来实现上述功能，格式：__Pragma("once")，参数是一个字符串，可以在宏中使用，因此会有更大的灵活性。



（4）变长参数的宏定义，以及___VA_ARGS__

  在C99标准中，用三个"..."来代表变长参数，取参数需要用

va_start / va_end / va_list / va_arg 组合来取参数，比如下面的代码：

	void Printf(const char* format, ...) {
		va_list ap;
		va_start(ap, format);
		vprintf(format, ap);
		va_end(ap);
	}

在使用高级特性的情况下，上述函数就可简化为：

```
#define Printf(format, ...) printf(format, __VA_ARGS__)
```



（5）宽字符连接

在C++11新标准之前，将榨字符（char）转为宽字符（wchar_t）是未定义的，但在新标准了，明确了在char和wchar_t进行连接时，会自动将char转为wchar_t再进行连接。



## （二）long long整



long long整型在1995年就有人提议写进C98标准，但被C++委员会拒绝，后来被写进C99标准，同时也被很多编译器支持，于是在C++11中正式被加入到标准中。

long long整型有**两种类型**：**long long**和无符号**unsigned long long**

C++11标准要求，long long整型在不同的平台上有不同的长度，但至少要有64位长度。我们在写常数时可以在末尾加LL（或ll）表明是long long整型。

比如：

​			`long long val1 = -9289398LL;`

​			`unsigned long long uval2 = 9283938ULL;`

同时，在C++11中，有很多与long long等价的类型，比如long long， signed long long，long long int，signed long long int都是等价的,无符号数也一样。

要了解平台上的long long大小，可以查看<climits>或<limits.h>进行查看，比如：



