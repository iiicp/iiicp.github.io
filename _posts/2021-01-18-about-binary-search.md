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


### 二分查找经典模式 

这个代码逻辑比较简单. 在[l, r]闭合区间中查找目标target

``` 
  /// 找到了返回该index，找不到返回-1
  int binary_search_normal(E *arr, int len, E target) {

    assert(arr && len > 0);

    int l = 0, r = len - 1;
    while (l <= r) {
      int mid = l + (r - l) / 2;
      if (target == arr[mid])
        return mid;
      else if (target < arr[mid])
        r = mid - 1;
      else
        l = mid + 1;
    }

    return -1;
  }
```  

### 二分查找变种-查找大于target的最小index 

首先要确定搜索区间。如果target大于数组所有元素，则返回数组最末index的后一个；如果target小于数组所有元素，则返回数组首元素index. 所以初始化区间在[0, len]，在这个闭合区间内是一定有解的。当左边==右边的时候，就找到了index.

``` 
  /// 查找大于target的最小的index
  /// 边界check: 如果target大于数组最大元素，返回数组最后一个元素的下一个index
  /// 如果target小于数组的最小元素，返回数组第一个元素
  int binary_search_upper(E *arr, int len, E target) {

     assert(arr && len > 0);

     int l = 0, r = len;
     while (l < r) {
       int mid = l + (r - l) / 2;
       if (target >= arr[mid])
         l = mid + 1;
       else
         r = mid;
     }
     return l;
  }
```   


### 二分查找变种-查找小于target的最大index 

同样首先确定区间。如果target大于数组所有元素，则返回数组最末元素index；如果target小于数组所有元素，则返回-1. 即在区间[-1, len-1]中进行搜索。通过下面代码也可以知道，在确定左边界的时候，要注意向上取整，否则当l和r相邻的时候，搜索区间没有移动。

``` 
  /// 查找小于target的最大的index
  /// 边界check: 如果target大于数组最大元素，取数组最后一个元素的index
  /// 如果target小于数组的最小元素，则返回-1
  int binary_search_lower(E *arr, int len, E target) {

    assert(arr && len > 0);

    int l = -1, r = len - 1;
    while (l < r) {
      int mid = l + (r - l + 1) / 2; /// 注意这里进行了加1操作，让其向上取整
      if (target <= arr[mid])
        r = mid - 1;
      else
        l = mid; /// 注意此处，当l和r相邻时，l没有更新，所以mid需要向上取整
    }
    return l;
  }
```   

### 其它变种 

后续四种变种是对前面两种变种的综合应用。核心就是先判断等于target的情况，然后求出大于target的最小index还是小于target的最大index。一并贴出。 

``` 
/// 查找大于等于target的index
  /// 如果找到了target，返回其最大的那个index
  /// 如果没有找到target，返回大于其值的最小的index
  int binary_search_upper_ceil(E *arr, int len, E target) {

    assert(arr && len > 0);

    int upperIndex = binary_search_upper(arr, len, target);
    if (upperIndex > 0 && arr[upperIndex - 1] == target)
      return upperIndex - 1;

    return upperIndex;
  }

  /// 查找大于等于target的index
  /// 如果找到了target，返回其最小的index
  /// 如果没有找到target，返回大于其值的最小的index
  int binary_search_lower_ceil(E *arr, int len, E target) {

    assert(arr && len > 0);

    int lowerIndex = binary_search_lower(arr, len, target);
    if (lowerIndex < len - 1 && arr[lowerIndex + 1] == target)
      return lowerIndex + 1;
    return binary_search_upper(arr, len, target);
  }

  /// 查找小于等于target的index
  /// 如果找到了target，返回其最大的index
  /// 如果没有找到target，返回小于target的最大的index
  int binary_search_upper_floor(E *arr, int len, E target) {

    assert(arr && len > 0);
    int upperIndex = binary_search_upper(arr, len, target);
    if (upperIndex > 0 && arr[upperIndex - 1] == target)
      return upperIndex - 1;
    return binary_search_lower(arr, len, target);
  }

  /// 查找小于等于target的index
  /// 如果找到了target，返回其最小的index
  /// 如果没有找到target, 返回小于target的最大的index
  int binary_search_lower_floor(E *arr, int len, E target) {

    assert(arr && len > 0);
    int lowerIndex = binary_search_lower(arr, len, target);
    if (lowerIndex < len - 1 && arr[lowerIndex + 1] == target)
      return lowerIndex + 1;
    return lowerIndex;
  }
``` 