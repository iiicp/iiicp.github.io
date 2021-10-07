---
layout: post
title: C++11 新特性
tags: modern cpp
excerpt: 复习现代c++的新特性
---



1> 模板双重右尖括号

```c++
/// old 
std::vector<std::list<int> >

/// c++11
std::vector<std::list<int>>
```



2> 空指针统一nullptr

```c++
/// old
char *c1 = NULL;
char *c2 = 0;

/// c++11
char *c3 = nullptr;

/// 解决指针的二义性问题
void func(char *c); 
void func(int c); 
// call func(int c);
func(NULL); 
// call func(char *c);
func(nullptr);
```



3> 一致性初始化,统一使用 {}

```c++
/// old
int a = 3;
Complex<float> c(3,5);
int arr[] = {1,2,3};

/// c++11
int a{3};
Complex<float> c{3,5};
int arr[]{1,2,3};

/// 优点
int a;
int a{}; ///会默认赋值为0
```



4> 自动类型推导auto

``` c++
/// old
std::vector<std::string> v{"study", "modern", "cpp"};
for (std::vector<std::string>::iterator it = v.begin(); it != v.end(); it++) {
}

/// c++11
for (auto it = v.begin(); it != v.end(); it++) {
}

// etc..
auto a = 12;
auto v = std::vector<int>{1,2,3,4};
```



5> 更便利的for循环 (for-range)

```c++
/// c++11
std::vector<std::string> v{"study", "modern", "cpp"};
for (auto &s : v){
}
```



6> 匿名对象 lambda

```c++
/// c++11
auto f = [](int a, int b) -> int {
  return a > b ? a : b;
}

int m = f(3, 5);

/// 找到容器中5的倍数
std::vector<int> number{1,2,5,12,20};
for_each(number.begin(), number.end(), [](int a) {
  	if (a % 5 == 0)
      std::cout << a << std::endl;
});
```



7> std::function与lambda

```c++
/// 使用std::function来封装函数，可以传递lambda，更方便。

#include <iostream>
#include <functional>

class OldCallBack {
public:
    /// typedef void(*DataCallBack)(char *data, int len);
    using DataCallBack = void(*)(char *data, int len);

    OldCallBack () {
        m_cb = nullptr;
    }

    void set_callback(DataCallBack cb) {
        m_cb = cb;
    }
    void send_callback() {
        if (m_cb) {
            m_cb(nullptr, 1024);
        }
    }
private:
    DataCallBack m_cb;
};


class NewCallBack{
public:
    /// 使用std::function包装函数，可传入 lambda, function object, member function...
    using DataCallBack = std::function<void(char *data, int len)>;

    NewCallBack () {
        m_cb = nullptr;
    }
    void set_callback(const DataCallBack &cb) {
        m_cb = cb;
    }
    void send_callback() {
        if (m_cb) {
            m_cb(nullptr, 1024);
        }
    }
    void member_func(char *data, int len) {
        std::cout << "new style: member func" << std::endl;
    }
private:
    DataCallBack m_cb;
};

/// old style 全局函数....
/// 为了能访问对象成员，还需要传递对象指针过来...
void callback_func(char *data, int len) {
    std::cout << "old style" << std::endl;
}

int main() {

    OldCallBack o_cb;
    o_cb.set_callback(callback_func);
    o_cb.send_callback();

    NewCallBack n_cb;
#if 1
    /// 传递lambda
    n_cb.set_callback([&n_cb](char *data, int len)->void {
        std::cout << "new style: lambda" << std::endl;
    });
#else
    /// 传递成员函数
    n_cb.set_callback(std::bind(&NewCallBack::member_func, &n_cb, std::placeholders::_1, std::placeholders::_2));
#endif
    n_cb.send_callback();

    return 0;
}
```



8> 萃取表达式类型decltype

```c++
/// 一般用于模块类型推导
int a = 3, b = 5;
decltype(a+b) c = 4;

template<typename T1, typename T2>
auto add(T1 a, T2 b)->decltype(a+b){
    return a+b;
}
```



9> explicit/final/override/default/delete

```c++
class Base {
public:
  /// 声明成 explicit，调用构造函数必须显示传参
  /// 防止隐式类型转换
  explicit Base(int n):m_num(n) {
  }
  
  /// default 关键字，告诉编译器生成默认的构造函数
  Base() = default;

  virtual ~Base() {
    std::cout << "~Base()" << std::endl;
  }

  /// delete 关键字，告诉编译器不要生成此函数
  Base(const Base&) = delete;
  Base& operator=(const Base&) = delete;
  Base(Base &&) = delete;
  Base& operator=(Base&&) = delete;

  virtual void print() {
    std::cout << "base print()" << "," << m_num << std::endl;
  }

private:
  int m_num;
};

class SubC : public Base {
public:
  explicit SubC(int m) : Base(m) {

  }
  ~SubC() {
    std::cout << "~SubC()" << std::endl;
  }
  
  /// override 关键字，使用在虚函数重写的时候
  /// 防止不小心写错了函数，此时编译器会报错
  
  /// final关键字，告诉编译器此需函数不能被子类重写了
  virtual void print() override final {
    std::cout << "subc print()" << std::endl;
  }

  /// 后缀const表示不会修改类成员属性
  void show() const {

  }

};

/// 此处使用final关键字，同时也告诉编译器 SubSubC 不能被继承了.
class SubSubC final: public SubC {
public:
  /// error 因为父类已经标注为 final了
//  virtual void print() override {
//    std::cout << __func__ << std::endl;
//  }
};
```



10> enum class

```c++
/// 使用枚举类，作用域更加安全。

enum class A{
  Int,
  Short,
  Float
};

/// 不会导致Int枚举值重复定义
enum class B{
  Int,
  Short,
  Float
};

/// c语言所有的枚举值在一个可见空间内，枚举值不能相等
enum C {
  Int,
  Short,
  Float
};

/// 会导致重复定义
//enum D {
//  Int,
//  Short,
//  Float
//};

class EnumClassTest{
public:
  EnumClassTest() : m_b(B::Int) {

  }
  void resetEnumValue(B b) {
    m_b = b;
  }
private:
  B m_b;
};
```



11> 智能指针shared_ptr/unique_ptr/week_ptr

```c++
 /// 智能指针最大优点是管理原生的堆内存，不用去手动释放堆内存

#include <memory>

class SmartPointerTest {
public:
  class Item {
  public:
    Item() {
      std::cout << "Item()" << std::endl;
    }
    ~Item() {
      std::cout << "~Item()" << std::endl;
    }
  };
  
  void testUnique() {
    std::unique_ptr<Item> u1(new Item());
    /// error c++14 提供
    ///std::unique_ptr<Item> u2 = std::make_unique<Item>();
    /// reset会释放之前的指针，始终只持有一份指针
    u1.reset(new Item());
    std::cout << "testUnique after()" << std::endl;
  }

  void testSharedPtr() {
    std::shared_ptr<Item> i1 = std::make_shared<Item>();
    /// 引用计数为1
    std::cout << i1.use_count() << std::endl;
    std::shared_ptr<Item> i2 = i1;
    /// 引用计数为2
    std::cout << i1.use_count()  << "," << i2.use_count() << std::endl;
    std::cout << "testSharedPtr after()" << std::endl;
    /// 引用计数为2
    std::weak_ptr<Item> w1 = i2;
    std::cout << w1.use_count() << "," << i2.use_count() << std::endl;
  }
};

```



12> 可变参数的模板 varidic template

```c++
/// 递归出口
void print() {

}

/// typename... 表示变参模块
/// Types表示一堆类型
/// sizeof... 表示求取这一堆类型的参数个数
/// print函数每次会分割成1个和other，这样最终就能调用到递归的出口
/// 此接口相对于下面接口是特化的接口
template<typename T, typename... Types>
void print(const T& firstArg, const Types&... args) {
  std::cout << firstArg << ", " << sizeof...(args) << std::endl;
  print(args...);
}

template<typename... Types>
void print(const Types&... args) {
  std::cout << "print all" << std::endl;
  std::cout << sizeof...(args) << std::endl;
}

```



13> 右值引用 && 万能参数 &&引用折叠&& 完美转发

```c++
/// 万能参数 Type&&
/// 完美转发 std::forward<Type>()

#include <string>
#include <list>

class Token{
public:
  Token(int type, std::string name, int line, int col)
      : m_type(type), m_name(name), m_line(line), m_col(col) {
    std::cout << "Token(int type, std::string name, int line, int col)" << std::endl;
  }

  Token(int type, std::string name)
      : m_type(type), m_name(name) {
    std::cout << "Token(int type, std::string name)" << std::endl;
  }

  Token(int type, int line, int col)
      : m_type(type), m_line(line), m_col(col) {
    std::cout << "Token(int type, int line, int col)" << std::endl;
  }

private:
  int m_type{};
  const std::string m_name{};
  int m_line{};
  int m_col{};
};

class TokenSequence{
public:
  TokenSequence() {}
  ~TokenSequence() {}


  /// 利用可变参数模板，封装可变参数
  /// 统一了调用token的构造函数
  template <typename... Types>
  void EmplaceBack(Types&&... args){
    token_sequence.emplace_back(std::make_shared<Token>(std::forward<Types>(args)...));
  }

  template <class T>
  void func(T& t) {
    std::cout << "void func(T& t)" << std::endl;
  }

  template <class T>
  void func(T&& t) {
    std::cout << "void func(T&& t)" << std::endl;
  }

  template <class T>
  void func_wrap(T&& t) {
    /// 引用变量t是左值
    ///func(t);
    /// 使用std::forward之后
    /// T -> int -> T&& 右值
    /// T -> int & -> int& && -> int&
    /// T -> int && -> int && && -> int &&
    func(std::forward<T>(t));
  }

private:
  std::list<std::shared_ptr<Token>> token_sequence;
};
```



14> 常量表达式关键字 constexpr

```c++
/// 编译器会在编译器尝试计算
/// 只能通过参数传递的信息，不能修改非局部变量
constexpr double func(double x, int n) const{
    double res = 1;
  	int i = 0;
  	while (i < n) {
      res *= x;
      ++i;
    }
  	return res;
}
```



15> sizeof/sizeof...

```c++
struct A {
  int data[100];
  int c;
};

/// 1, 直接可以sizeof对象的某个成员的大小
std::cout << sizeof(A::data) << std::endl;

template<typename... Types>
void print(const Types&... others) {
/// 2, 表示获取可变类型的个数
  std::cout << sizeof...(others) << std::endl;
}
```



16> 新增的标准库

```c++
/// 数据结构
#include<unordered_map>
#include<unordered_set>
#include<array>
#include<forward_list>
#include<tuple>
/// 正则表达式库
#include<regex>
/// 线程库（不用每个平台单独实现了）
#include<thread>
/// 时间库
#include<chrono>
```



17>标准库chrono使用

```c++
/// 0, 首先选择时钟，一般是system_clock或者steady_clock
/// 1. 获取开始时间点
///     time_since_epoch 表示获取到距离1970年的时间ms
/// 2. 获取结束时间点
///      时间点间可以进行time_point_cast<>
/// 3, 时间点的差值就是 duration
///      duration之间也可以进行转换,使用duration_cast<>
///    最后打印的时候都是通过count()来做

  void sum10000() {
    float sum = 1;
    for (int i = 0; i < 10000; ++i) {
      for (int j = 0; j < 10000; ++j) {
        sum += (i+j+1);
      }
    }
  }
  void testSystemClock() {
    using namespace std::chrono;

    /// 获取开始时间点
    time_point<system_clock> start = system_clock::now();
    sum10000();
    ///std::this_thread::sleep_for(std::chrono::seconds(2));
    /// 获取结束时间点
    time_point<system_clock> end = system_clock::now();

    duration<double> elapsed_s = (end - start);
    duration<double, std::milli> elapsed_ms =  end - start;
    duration<double, std::micro> elepsed_mics = end - start;
    duration<double, std::nano> elepsed_nanos = end - start;
    std::cout << "Elapsed time: " << elapsed_s.count() << " s" << std::endl;
    std::cout << "Elapsed time: " << elapsed_ms.count() << " ms" << std::endl;
    std::cout << "Elapsed time: " << elepsed_mics.count() << " us" << std::endl;
    std::cout << "Elapsed time: " << elepsed_nanos.count() << " ns" << std::endl;

    /// 获取时间戳
    /// 默认是us
    std::cout << start.time_since_epoch().count() << std::endl;
    /// 时间点可以转换成其它单位，eg 转成了second
    auto s = time_point_cast<seconds>(start);
    std::cout << s.time_since_epoch().count() << std::endl;
  }
```

