---
layout: post
title: 一些C编译器实现的比较
tags: c语言
excerpt: 深入学习c
---



最近在阅读一些c99/c11编译器实现的代码，这些代码各有优缺点，先总结如下。

| 仓库          | 特点                                                         |
| ------------- | ------------------------------------------------------------ |
| wgtcc(c11)    | c++11/pre/lex/parse/sema/codegen/64位;type和ast的抽象层次稍有欠缺 |
| axcc(c11)     | c++11/pre/lex/parse/sema;缺失代码codegen;type/ast抽象层次不错，代码质量高 |
| cld(c99)      | c++17/pre/lex/parse/sema/codegen/llvm;使用了第三方;虽然使用了很多新特性，但是代码并不简洁，甚至比较难以阅读，代码也不够严谨 |
| clang2.7(c99) | c++/pre/lex/parse/sema/codegen/llvm;最权威的;代码文件多，且包含一些自动生成的机制，难以阅读；使用了很多模块特性和llvm数据结构，难以拆解。 |
| ucc(c89)      | c89/lex/parse/sema/ir/codegen/32位;比较完整的实现，命名比较规范，代码抽象层次不高，不够高内聚，也有比较多的补丁。可惜后端是个32位。 |
| lcc(c89/c99)  | c89/lex/parse/sema/codegen/32位; 属于比较早期的实现，代码命名过于不规范，难以阅读，后端采用生成工具，难以阅读。只能用于帮助理解概念。 |
| chibicc(c11)  | c11/pre/lex/parse/sema/codegen/64；比较完整的实现，代码抽象层次不高，对于后端代码生成，值得比较好好学习。 |
| subc          | c89/lex/parse/sema/codegen/32位; 代码实现没有什么技巧吧，纯粹是为了实现编译器，完整性不如ucc。 |
|               |                                                              |

我的述求：

一：能够锻炼编程能力，所以我倾向于选择抽象能力高的，具备一定的设计能力，我选择c++实现的

二：阅读编译器代码，我希望其实现比较完整，前端的语言解析完整，后端倾向于64位

从业界来说，最好的编译器实现当然是clang，可惜实在是过于庞大了，即使是早期版本（driver/数据结构/模板/tablegen/cmakelists构建）。退而求其次，我觉得axcc/wgtcc/chibicc/ucc是比较好的选择.

**axcc为主, wgtcc补充，解决我的第一点要求。**

**chibicc/ucc补充代码生成和编译知识，这两个都实现的比较完整，解决我的第二点需求。**

我是隔一段时间就开始挑战clang...，每次都是失败，确实是过于庞大！！！