---
layout: post
title: C++14 新特性
tags: modern cpp
excerpt: 复习现代c++的新特性
---



#### 对C++11的一个补充，没有太多新的特性

1> 增强constexpr

```c++
/// 说明c++更加推荐constexpr，将更多事情放在编译时计算

/// c++14 ok，可以使用函数局部变量
  constexpr int sum(int n) {
    int ret = 0;
    for (int i = 0; i <= n; ++i) {
      ret += i;
    }
    return ret;
  }

/// c++14 ok，可以多处return来返回值
  constexpr int func(bool flag) {
    if (flag) return 1;
    else return 0;
  }
```



2> 增加std::make_unique方法

```c++
/// 增加std::make_unique来构建unique对象，填补c++11

/// c++11
///std::unique_ptr<Item> i(new Item());

/// c++14
/// 新增了make_unique 
std::unique_ptr<Item> it1 = std::make_unique<Item>();
std::unique_ptr<Item> it2 = std::make_unique<Item>();
std::unique_ptr<Item> it3(std::move(it2));
std::cout << "after..." << std::endl;
```



3> 新增deprecated标签

```c++
/// 比较实用的特性
/// 可以标注某个class已经被废弃
struct [[deprecated]] A {
  int a;
};

/// 再次使用编译器会报错
A a;
```



4> 数值常量支持分割符

```c++
int b = 0b1110'1111;
double d = 12'345.1'234'567;
```



5> 支持给字符串加引号

```c++
#include <iomanip>

std::string s = "study modern cpp";

/// 会打印带有引号的字符串
std::cout << std::quoted(s) << std::endl;
```



6> thread库新增共享锁，支持读多写少的场景

```c++
#include <thread>
#include <shared_mutex>

class SharedTimeTest {
public:
  SharedTimeTest() {}
  ~SharedTimeTest() {}

  void numberIncrement() {
    std::lock_guard<std::shared_timed_mutex> lock(m_mutex);
    for (int i = 0; i < 10000; ++i)
      ++value;
  }

  void numberDecrement() {
    std::lock_guard<std::shared_timed_mutex> lock(m_mutex);
    for (int i = 0; i < 10000; ++i)
      --value;
  }

  unsigned int numberPrint() {
    std::lock_guard<std::shared_timed_mutex> lock(m_mutex);
    return value;
  }

private:
  /// 读写锁互斥量
  mutable std::shared_timed_mutex m_mutex;
  unsigned int value;
};
```

