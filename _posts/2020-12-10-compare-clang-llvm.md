---
layout: post
title: llvm2.7可能是便于研究的一个版本了  
tags: clang/llvm
excerpt: 比较llvm 
---  


### 比较不同的llvm版本   

llvm2.6，对应于clang1.0，2009.10.23，这个版本使用gcc4.4能进行编译，但是我在host编译成功之后，执行clang xx.c是失败的.
llvm2.7，对应于clang1.1，2010.4.27，这个版本4.4能编译,而且尝试使用clang命令，能正常输出可执行文件，这个版本适合研究！！
llvm2.8，对应于clang2.8，2010.10.05，这个版本可以用4.6进行编译了。gcc从4.7开始支持c++11。
llvm2.9，对应于clang2.9，2011.4.06，这个版本不能使用4.7编译，4.6依然可以。    

llvm2.8和2.9源码，对比llvm2.6和2.7复杂了不少，从clang的版本命名可以看出，2.6和2.7分别叫做clang1.0，clang1.1,后面clang的版本都是对接llvm了，对我来说，我目前只想研究c编译器，所以看来llvm2.7是一个不错的方案。

```
#llvm2.7 编译产物

-rwxr-xr-x 1 root root  73447916 Dec 11 12:24 bugpoint*
-rwxr-xr-x 1 root root 214815933 Dec 11 12:29 clang*
lrwxrwxrwx 1 root root        20 Dec 11 12:32 clang++ -> /usr/local/bin/clang
-rwxr-xr-x 1 root root  87024723 Dec 11 12:23 llc*
-rwxr-xr-x 1 root root  88957656 Dec 11 12:23 lli*
-rwxr-xr-x 1 root root  20050263 Dec 11 12:23 llvm-ar*
-rwxr-xr-x 1 root root  22069220 Dec 11 12:23 llvm-as*
-rwxr-xr-x 1 root root   2461811 Dec 11 12:24 llvm-bcanalyzer*
-rwxr-xr-x 1 root root     19146 Dec 11 12:23 llvm-config*
-rwxr-xr-x 1 root root  19157421 Dec 11 12:23 llvm-dis*
-rwxr-xr-x 1 root root  24185560 Dec 11 12:23 llvm-extract*
-rwxr-xr-x 1 root root  51131134 Dec 11 12:23 llvm-ld*
-rwxr-xr-x 1 root root  24054468 Dec 11 12:23 llvm-link*
-rwxr-xr-x 1 root root  83375035 Dec 11 12:23 llvm-mc*
-rwxr-xr-x 1 root root  19891005 Dec 11 12:23 llvm-nm*
-rwxr-xr-x 1 root root  22007319 Dec 11 12:23 llvm-prof*
-rwxr-xr-x 1 root root  19877976 Dec 11 12:23 llvm-ranlib*
-rwxr-xr-x 1 root root     11594 Dec 11 12:24 llvm-stub*
-rwxr-xr-x 1 root root  70917564 Dec 11 12:23 opt*

```

``` 
#llvm2.8 编译产物  

-rwxr-xr-x 1 root root  75404985 Dec 10 16:05 bugpoint*
-rwxr-xr-x 1 root root 265004580 Dec 10 16:11 clang*
-rwxr-xr-x 1 root root  96878976 Dec 10 16:04 llc*
-rwxr-xr-x 1 root root  94120079 Dec 10 16:05 lli*
-rwxr-xr-x 1 root root  19543393 Dec 10 16:04 llvm-ar*
-rwxr-xr-x 1 root root  21578174 Dec 10 16:04 llvm-as*
-rwxr-xr-x 1 root root   2584827 Dec 10 16:05 llvm-bcanalyzer*
-rwxr-xr-x 1 root root     19272 Dec 10 16:04 llvm-config*
-rwxr-xr-x 1 root root  21449182 Dec 10 16:05 llvm-diff*
-rwxr-xr-x 1 root root  18688778 Dec 10 16:04 llvm-dis*
-rwxr-xr-x 1 root root  26898440 Dec 10 16:05 llvm-extract*
-rwxr-xr-x 1 root root  51629916 Dec 10 16:05 llvm-ld*
-rwxr-xr-x 1 root root  23718188 Dec 10 16:05 llvm-link*
-rwxr-xr-x 1 root root  90465454 Dec 10 16:04 llvm-mc*
-rwxr-xr-x 1 root root  19373395 Dec 10 16:04 llvm-nm*
-rwxr-xr-x 1 root root  21243814 Dec 10 16:05 llvm-prof*
-rwxr-xr-x 1 root root  19386375 Dec 10 16:04 llvm-ranlib*
-rwxr-xr-x 1 root root     11422 Dec 10 16:05 llvm-stub*
-rwxr-xr-x 1 root root  72948605 Dec 10 16:04 opt*

```   

``` 
#llvm2.9 编译产物   

drwxr-xr-x 2 root root      4096 Dec 10 15:45 ./
drwxr-xr-x 6 root root      4096 Dec 10 15:45 ../
-rwxr-xr-x 1 root root  79136088 Dec 10 15:34 bugpoint*
lrwxrwxrwx 1 root root         9 Dec 10 15:45 clang -> clang-2.9*
lrwxrwxrwx 1 root root         5 Dec 10 15:45 clang++ -> clang*
-rwxr-xr-x 1 root root 309492070 Dec 10 15:42 clang-2.9*
-rwxr-xr-x 1 root root 103077940 Dec 10 15:34 llc*
-rwxr-xr-x 1 root root 101943328 Dec 10 15:34 lli*
-rwxr-xr-x 1 root root  17421564 Dec 10 15:34 llvm-ar*
-rwxr-xr-x 1 root root  21546269 Dec 10 15:34 llvm-as*
-rwxr-xr-x 1 root root   2812477 Dec 10 15:34 llvm-bcanalyzer*
-rwxr-xr-x 1 root root     19254 Dec 10 15:33 llvm-config*
-rwxr-xr-x 1 root root  19224900 Dec 10 15:34 llvm-diff*
-rwxr-xr-x 1 root root  16581618 Dec 10 15:34 llvm-dis*
-rwxr-xr-x 1 root root  27054641 Dec 10 15:34 llvm-extract*
-rwxr-xr-x 1 root root  74094263 Dec 10 15:34 llvm-ld*
-rwxr-xr-x 1 root root  23659524 Dec 10 15:34 llvm-link*
-rwxr-xr-x 1 root root       825 Dec 10 15:27 llvm-lit*
-rwxr-xr-x 1 root root  94422561 Dec 10 15:34 llvm-mc*
-rwxr-xr-x 1 root root  17584335 Dec 10 15:34 llvm-nm*
-rwxr-xr-x 1 root root  93675780 Dec 10 15:34 llvm-objdump*
-rwxr-xr-x 1 root root  20688854 Dec 10 15:34 llvm-prof*
-rwxr-xr-x 1 root root  17259509 Dec 10 15:34 llvm-ranlib*
-rwxr-xr-x 1 root root     11422 Dec 10 15:34 llvm-stub*
-rwxr-xr-x 1 root root   2357436 Dec 10 15:34 macho-dump*
-rwxr-xr-x 1 root root  76812557 Dec 10 15:34 opt*
-rwxr-xr-x 1 root root  19442039 Dec 10 15:28 tblgen*  
```   
