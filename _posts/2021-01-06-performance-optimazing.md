---
layout: post
title: 计算性能优化
tags: SSE NEON
excerpt: about performance-optimazing
---  

### 前言 

计算性能优化，在算法复杂度不变的前提下，能够充分利用并行指令和缓存来做优化，加速运算。
性能优化是一个无穷无尽的过程，是极度需要耐心和技巧的过程.

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

[openblas作者的一个报告]https://www.leiphone.com/news/201704/Puevv3ZWxn0heoEv.html  

关于原生c++实现矩阵乘法和矩阵和向量乘法与openblas的对比，可以参考我的github仓库. [openblas的优化对比](https://github.com/iiicp/project-arrange/tree/master/performance-optimazing)  

下面列出来关于使用openblas的三个函数    
``` 
  /// y = alpha * x + y; (小规模向量会负优化，至少要大于512)
  void VecAddVec_blas(Vec<float> &x, Vec<float> &y, float alpha) {
    assert(x.row == y.row);
    int n = x.row;
    cblas_saxpy(n, alpha, x.data, 1, y.data, 1);
  }

  /// C = alpha * A * b + beta * C
  Vec<float> MatMulVec_blas(Matrix<float> &A, Vec<float> &b) {
    assert(A.col == b.row);

    Vec<float> c(A.row);

    int M = A.row, N = A.col;
    float alpha = 1.0f, beta = 0.0f;
    int lda = A.col;
    cblas_sgemv(CblasRowMajor, CblasNoTrans, M, N, alpha, A.data, lda, b.data,
                1, beta, c.data, 1);

    return c;
  }

  /// C = alpha * A * B + beta * C
  Matrix<float> MatMulMat_blas(Matrix<float> &A, Matrix<float> &B) {
    assert(A.col == B.row);

    Matrix<float> C(A.row, B.col);

    int M = A.row, N = B.col, K = B.row;
    float alpha = 1.0f, beta = 0.0f;
    int lda = A.col, ldb = B.col, ldc = B.col;
    cblas_sgemm(CblasRowMajor, CblasNoTrans, CblasNoTrans, M, N, K, alpha,
                A.data, lda, B.data, ldb, beta, C.data, ldc);

    return C;
  }
```  

以下是测试结果，矩阵乘法和矩阵乘向量是一个巨大的优化!!!  

``` 
C = alpha * A * B + beta * C: [256,128] x [128,64]
native cpp time: 9.639ms
openblas time: 0.393ms

C = alpha * A * b + beta * C: [256,161] x [161, 1]
native cpp time: 0.403ms
openblas time: 0.017ms

y = alpha * x + y: [512,1] + [512,1]
native cpp time: 0.009ms
openblas time: 0.004ms

``` 

### LSTM算子实现

LSTM的逻辑计算结构是下图 

![lstm的逻辑结构图](https://pic3.zhimg.com/80/v2-074e25fdd8f18378e9f7d2f48d9259ca_1440w.jpg) 

遗忘门  

forget_gate = tf.sigmoid(tf.matmul(Xt, W1) + tf.matmul(H(t-1), W2) + b12);     

信息增强门   

input_gate = tf.sigmoid(tf.matmul(Xt, W3) + tf.matmul(H(t-1), W4) + b34);   
middle_memory = tf.tanh(tf.matmul(Xt, W5) + tf.matmul(H(t-1), W6) + b56); 

输出门  

output_gate = tf.sigmoid(tf.matmul(Xt, W7) + tf.matmul(H(t-1), W8) + b78);

最终输出   

Ct = C(t-1) * forget_gate + input_gate * middle_memory; 
Ht = tf.tanh(Ct) * output_gate;
yt = tf.softmax(Ht) /// 未使用到 

从上面的计算过程中，可以发现 Xt * W(1,3,5,7) + b(12,34,56,78) 是可以合并成 向量x矩阵+偏置也就是全连接层。
而H(t-1) * W(2,4,6,8)可以向量x矩阵来合并优化。再将第一步和第二步求和，即可以得到所有激活函数之前的值。  


参考链接    
https://zhuanlan.zhihu.com/p/76174753      
https://www.zhihu.com/question/41949741/answer/318977452


### CNN算子实现

整体理解起来比lstm要简单，cnn结构有如下基本概念: 

1. padding   
表示在input数据左右填充一些0，改变输出的大小。 

2. stride 
卷积核位置每间隔多少来取数据, 默认stride是1. stride同样也会改变输出大小。 

假设卷积核大小为(FH, FW), 输入大小(H, W)，那么padding和stride之后，输出大小为(OH, OW)有如下关系    
OH = (H+2P-FH)/S + 1;    
OW = (W+2P-FW)/S + 1;   

多于多个卷积核的场景，权重数据的排布 (output_channel, input_channel, FH, FW)，比如通道数为3，大小为5x5的滤波器有20个时，可以写成(20, 3, 50, 50)   

3. 池化层  

池化层的特征，无学习参数，通道数不变化。 
池化层和卷积层不同，没有要学习的参数，池化只是从目标区域中取最大值（或者平均值），所以不存在要学习的参数 
经过池化运算，输入数据和输出数据的通道数不会发生变化，计算是按照通道独立进行的。

