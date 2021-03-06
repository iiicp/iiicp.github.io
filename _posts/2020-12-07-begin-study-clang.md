---
layout: post
title: 开始clang/llvm学习
tags: clang/llvm
excerpt: 这些天都在抽离clang2.6的lex代码
---

### clang2.6的总体印象

花了两三周时间来抽离clang第一个释放版本的代码。
clang是基于llvm的一个c/c++/oc的前端项目, 负责用户输入的命令处理(Driver), 这些语言的预处理、词法分析、语法分析、语义分析、
调用llvm的模块代码进行代码生成，另外还要提供用户的输出，比如原生token输出、预处理token输出、预处理输出
（是否保留注释、是否保留line指令，是否保留define指令呀等）、HTML和XML格式输出、语法树输出等，简而言之就是
尽量智能化的处理用户的action、尽量多元化的输出内容给用户。中间的内容就是传统的编译原理内容（词法包含预处理，语法、语义）
clang是同时支持三种语言，所以提出来了语言配置选项来进行扩展，clang同时也处理用户要指定生成某一个平台，所以也自动配置了target信息，clang的diagnoist信息是一个依赖tablegen来生成的，虽然对源码阅读者不友好，但是对clang developer来说要更好，
写更少的代码来生成和扩展更多的诊断信息。预处理和词法分析用不同的类来抽象，同时又整合在一起是非常优秀的。对于文件的抽象也
是非常好的，虽然新手阅读起来痛苦，但是也逐渐去适应的。另外clang采用了cmake来构建，也是一个不错的构建实践。

### 通过clang能学习到什么

一, 大型项目的架构设计，如何分离模块，如何抽象可变，cmake的构建手段。clang对接不同的系统，关于系统的搜索路径，
系统调用等都是做了平台抽象隔离的，这个是llvm去做的.        
二, c++的实践，学习c++的代码编写，以及模板编程相关.    
三, 编译原理的最佳实践吧，如何有效的组合词法、语法、语义等内容.    
四, 我认为非常重要的一点，如何给出优秀的输出，比如预处理、token、语法树等，这些对于数据信息的掌控是非常重要的，
当然也是依托于优秀的数据结构设计.    
五, 数据结构封装，因为编译器项目是从源码字符串读入开始，这些读入的字符串都是在内存中了，是要尽量减少内存拷贝的，    
所以llvm抽象了stringRef，stringMap，DenseMap等等数据结构，c++标准的数据结构的元素都是需要拷贝的，因此llvm这些
数据结构的设计初衷就是最少的内存开销，clang前端速度要尽量快。通过clang对文件的抽象就能看出来了，尽量使用cache的
思路去做.

### 计划步骤是什么

一, 编译llvm，使用cmake来引入库和头文件
通过这几周抽离lex代码的过程，我理会到要学习clang，就要先隔离llvm会比较好，特别是到语法分析、语义分析、
代码生成等可能依赖的llvm的东西会更多了，而这些东西都是比较经典的内容和clang混在一起只会更加加大学习困难度。
所以我打算将llvm编译成库，然后让我的裁剪版本clang去调用，clang基于2.6，llvm可能基于比较新的版本。
通过这种模式也能学习，如何利用llvm来开发新项目。所以第一个难点是编译llvm，利用cmake来引入。

二, clang从不同的模块开始研究，目前我想的是可以从driver开始。

