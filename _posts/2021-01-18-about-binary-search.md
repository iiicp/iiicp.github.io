---
layout: post
title: 关于二分查找及其变种
tags: datastruct
excerpt: binary search
--- 

### 前言 

二分查找可以说是基于比较的最快的查找的算法了。二分查找及其变种大概是如下几类。

代码参考[我的仓库](https://github.com/iiicp/datastruct-algorithm/blob/master/BinarySearch/BSearch.h)

1. 查找等于target的index，找不到返回-1.
2. 查找大于target的最小的index
3. 查找小于target的最大的index
4. 查找大于等于target的index，如果找到等于target的index，返回其最大的index，否则返回大于target的最小index.
5. 查找大于等于target的index，如果找到等于target的index，返回其最小的index，否则返回大于target的最小index.
6. 查找小于等于target的index，如果找到等于target的index，返回其最大的index，否则返回小于target的最大index.
7. 查找小于等于target的index，如果找到等于target的index，返回其最小的index，否则返回小于target的最大index. 

其中后面4种变种依赖于第2、3两种。下面一一进行说明。 


