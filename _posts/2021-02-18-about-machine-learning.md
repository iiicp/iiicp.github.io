---
layout: post
title: 关于机器学习
tags: machine learning
excerpt: machine learning
---  

### 前言

这个假期主要学习了下机器学习相关的算法。机器学习主要用来解决分类和回归问题，这一类问题是传统算法不太好解决的，比如预测某张图片是猫还是狗，预测某个区域的房价的价格。机器学习解决这类问题是依托训练数据拟合出一个模型然后对新数据进行预测。这种解决问题的思路有些像人脑，所以机器学习也叫做人工智能。

### 机器学习的大体流程 

其实通过sklearn包的封装，我们就能知道机器学习的大体流程了。

1. 首先是进行数据预处理。一般是训练集和测试集分离，这个要随机打乱。   
2. 其次可能要做归一化处理。如果涉及到距离，那么数据的量纲要保持一致，机器学习一般是做均值方差归一化。   
3. 用训练集去拟合出模型。   
4. 用测试集去预测。可能是预测一个label，也可能是一个连续的值（比如直线或曲线）   
5. 对效果进行打分。回归问题使用r2_score进行评测，分类问题使用f1_score来进行评测  
6. 一些可视化处理。
7. 过拟合和欠拟合的处理。这一步是训练过程中反复进行的，进行超参数、数据的一些处理。    

回归问题包含线性回归，多项式回归。线性回归是预测一条直线，线性回归进行向量化处理之后就是矩阵和向量的乘积，矩阵表示的是预测数据矩阵，向量表示的是拟合出的线性回归的参数，向量的维度和特征的维度是一样的。其实深度学习也包含大量的矩阵和向量的乘积，也就是前向预测过程吧。有些问题并不是线性可分的，而是需要用曲线来进行拟合，这就涉及到多项式回归了。机器学习解决多项式回归非常巧妙，是将多项式的高阶项都看成线性回归的特征，其原理就是对于线性不可分问题，在高维度就会变成可分的。所以机器学习的预处理中会包含生成高阶项的特征，比如是要拟合出**x + x^2 + b**这条曲线，其中x表示向量，那么特征项就是x和x^2，按列拼接起来，所以多项式
回归就是特征项更多线性回归。解决回归问题一般不会用解析解，因为可能没有解析解，而是会用梯度下降法去求解。使用梯度下降法，数据的归一化是很重要的，不然梯度向量会非常大，导致不收敛。引入梯度下降法就会学习率和batch size、迭代次数等超参数。

分类问题包含二分类和多分类。常用的算法有knn、逻辑回归、svm。knn思路是相当简单，对于预测数据计算其与每一个训练数据的距离，然后选出最近的k个距离，根据这k个点的label，来进行划分，label相同最多的就是它的预测值。很明显knn的预测过程比较慢，但是knn天然是支持多分类的，然后对于一些数据集合，效果还很不错。knn里面有一些超参数，比如k、距离等。sklearn里面封装了搜索超参数的方法，对于算法工程师来说很有帮助。逻辑回归是在线性回归的基础上加了一个激活函数sigmoid，对于线性回归预测的值压缩到0到1之间的概率值，然后根据0.5来进行分类。线性回归+sigmoid在深度学习里面非常常见。逻辑回归也能解决非线性问题，原理就是线性回归+多项式特征提取，然后进行sigmoid操作。svm的思路是找到一个决策边界，让离这个决策边界的数据点距离决策边界尽可能远，距离决策边界最近的点称之为支撑向量。svm是一个带条件的优化问题。 

评价指标。这个是很重要的，对于分类问题要综合多种指标，因为数据有可能极度有偏，比如预测某个病人是否患有癌症，这是非常小概率事件，不能仅仅依靠分类准确率来进行评价。所以分类问题引出来了混淆矩阵的概念，有精确率、召回率等等指标，现在sklearn封装的是f1_score来进行分类打分。回归问题采用r2_score来进行打分。

### 总结  

首先要对问题分类，是回归问题还是分类问题。一般来说是监督学习。比如语音增强的降噪问题，被认为是一个分类问题，有点像逻辑回归，产生一些概率值。其次就是数据的处理，比如训练集测试集分离、数据归一化、数据label标注等，继而采用求解方法，一般是定义损失函数、然后进行求导、采用梯度下降来进行求解，最后就是预测打分和调参了。

机器学习知识点比较多，做一个程序员真心不容易。


