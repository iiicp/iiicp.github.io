---
layout: post
title: 计算性能优化
tags: SSE NEON
excerpt: about performance-optimazing
---  

### 前言 
计算性能优化，在算法复杂度不变的前提下，能够加速运算。
SIMD指令，简称单指令多数据流，一个指令能够操作多个数据流，大大减少了计算的
指令数目，达到加速运算的目的。

1. openblas加速运算
2. NEON指令 
3. SSE指令 
4. 定点化操作
5. 查表优化，空间换时间
6. 位运算优化
7. 数值计算优化，牛顿迭代，最小二乘法
8. 时频域变换，时域卷积等于频域的乘积
9. fft运算优化

### openblas的加速   

在移动端部署神经网络模型，由于手机一般是高通芯片，主频都比较高，再加上模型比较小的话，是可以采用openblas来进行加速运算矩阵x向量，以及向量的内积运算。 

https://www.leiphone.com/news/201704/Puevv3ZWxn0heoEv.html

### NEON指令优化矩阵乘

### SSE指令优化矩阵乘

### 定点化原理
