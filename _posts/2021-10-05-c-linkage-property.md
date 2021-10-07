---
layout: post
title: c的linkage Property
tags: c语言
excerpt: 深入学习c
---


如果不是看c编译器的实现，c的linkage property我应该关注不到。这点主要是体现在c的语义分析中。

c的linkage property主要是为了解决多个文件或者单个文件中同名的**全局变量和函数**，如何映射到内存。注意不包含局部变量，这里的原因是局部变量是分配到栈上的，属于自动变量，在块外是不可见的，也即不存在这个linkage过程，因此局部变量是不能重名的，不同的变量名表示栈上不同的地址。局部变量的linkage都是属于none.

c定义了如下规则，linkage包含intern/extern/none，同时根据是否初始化分为尝试性定义和定义。默认声明的全局变量，例如：

```c
int a; /// 默认是extern property
```

是属于extern的，也即外部可见的。

| 事例                                | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| int a = 1;                          | extern，外部文件可以访问，多个外部文件不可同时定义int a = x; |
| static int a = 1;                   | intern，外部文件不可访问，多个外部文件可以同时声明和定义static int a = x; |
| {int a;}                            | none，block外不可访问，不存在linkage，block内部不可重名      |
| extern int a;                       | extern第一次声明a，表示a属于外部链接。声明同时定义，编译器会有warning.  比如exern int a = 2; |
| static int a; extern int a;         | 此时extern并不是第一次声明a了，extern在此处只表示a是一个声明，其linkage不会改变，在此处仍然为intern. |
| extern int a; static int a;         | extern int a;表示a是extern，后续的static int a；表示a是intern，此时会出现语义冲突，编译器会报错。 |
| int a;  int a = 1; int a;           | 多个extern变量，尝试性定义，当遇到定义的时候，可以合并到第一个定义处，即此处多个同名a变量，指向内存中的同一处单元。 |
| static int a = 1; static int a = 1; | 多个intern变量，尝试性定义，当遇到定义的时候，可以合并到第一个定义处，即此处多个同名a变量，指向内存中的同一处单元。 |
|                                     |                                                              |

我个人猜测之所以让多个同名的全局变量能够合并，也是为了最大可能的避免编译冲突。

所以基于linkage，对于c语言编程，尽量使用static变量，来替代默认的全局变量。如果一定要外部文件可知，在命名上尽量唯一点，避免出现多处定义。

