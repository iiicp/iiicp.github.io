---
layout: post
title: C++17 新特性
tags: modern cpp
excerpt: 复习现代c++的新特性
---


1> 结构化绑定

```c++
/// 可以将public的属性成员绑定到变量上.

std::map<int, std::string> m = {
  {1, "study"},
  {2, "modern"},
  {3, "cpp"}
};
/// old
for (auto &elem : m) {
  std::cout << elem.first << "," << elem.second << std::endl;
}

/// c++ 17
/// 将std::pair绑定到[key, value]
for (const auto &[key, value] : m) {
  std::cout << key << ", " << value << std::endl;
}

/// 插入返回值绑定
auto [pos, ok] = m.insert({1, "bstudy"});
if (!ok) {
  const auto&[key, value] = *pos;
  std::cout << key << " is exist!!!" << "value: = " << value << std::endl;
}

/// std::array && std::tuple 都可以
std::array<int, 4> arr{1,2,3,4};
auto &[a, b, c, d] = arr;

std::tuple tu{1,2,3,4,5};
/// 先绑定到这些变量
auto &[a,b,c,d,e] = tu;
/// std::tie可以聚合这些变量：前提是这些变量已经被定义过
std::tie(a,b,c,d,e) = tu;
```



2> 带初始化语句的if和switch

```c++
/// 作用是能够缩小变量的作用域范围

void testIf() {
    std::map<std::string, int> coll;

    ///init的变量能够在所有的if分支都能访问，包括else
    if (auto[pos, ok] = coll.insert({"new", 42}); !ok) {
      const auto& [key, val] = *pos;
      std::cout << "already there: " << key << std::endl;
    }else {
      auto &[key, value] = *pos;
      ///...
    }
  }

void testSwitch(std::string path) {
  /// c++17 新增的filesystem
    namespace fs = std::filesystem;
    /// 也可以声明多个同类型变量
    /// fs::path p{path}, p2{path};
    switch (fs::path p{path}; status(p).type()) {
    case fs::file_type::not_found:
      std::cout << p << "not found" << std::endl;
      break;
    case fs::file_type::directory:
      std::cout << p << ":\n";
      for (const auto& e : fs::directory_iterator{p}) {
        std::cout << "-" << e.path() << '\n';
      }
      break;
    default:
      std::cout << p << " exists\n";
      break;
    }
```



3> 内联变量

```c++
/// 解决在类的头文件定义变量，其它cpp文件导入头文件的链接错误问题
/// 推荐!!!!

class InlineVaribleTest{
public:
  /// inline 就可以在头文件中定义 全局变量了
  /// c++11 只允许在类内初始化const
  inline static std::string s{"first"};
  const int a = 5;
};

/// inline 可以定义对象
/// 其它cpp文件导入这个头文件，不会导致重复定义的问题
inline InlineVaribleTest vtvt;
///std::string InlineVaribleTest::s{"ok"};
```



4> 类模块参数自动推导

```c++
/// c++11/14
std::vector<int> v{1,2,3,4}

/// c++17，可以省略模块类型声明
std::vector v{1,2,3,4};
```



5> 编译期if constexpr

```c++
/// 在编译期求值if内容，如果匹配，else的代码会进行丢弃处理

/// 如果不是if constexpr，会导致编译出错
/// 原因是：之前的模块编译是需要每个分支都能进行正常匹配。
/// 例如：asString(2), 则第一个分支return x;会导致编译失败，类型不匹配。
template <typename T>
std::string asString(T x)
{
  /// 如果传入内容是string，返回传入的内容
  if constexpr(std::is_same_v<T, std::string>) {
    return x;
  }
  /// 如果是数值类型则进行数值转字符串
  else if constexpr(std::is_arithmetic_v<T>) {
    return std::to_string(x);
  }
  else {
    return std::string(x);
  }
}

/// 完美返回泛型值
/// decltype(auto) 不能推导为void，因为void是不完全类型
/// 编译期if constexpr的一处妙用

#include <functional> // for std::forward()
#include <type_traits> // for std::is_same<> and std::invoke_result<>

template<typename Callable, typename... Args>
decltype(auto) call(Callable op, Args&&... args)
{
  if constexpr (std::is_void_v<std::invoke_result_t<Callable, Args...>>) {
    // 返回类型是void
    op(std::forward<Args>(args)...);
    return;
  }else {
    // 返回值类型不是void
    decltype(auto) ret{op(std::forward<Args>(args)...)};
    return ret;
  }
}
```



6> 折叠表达式(... op args)

```c++
/// 很牛的语言特性!!!

/// c++17以前的写法
/// 递归出口
template<typename T>
auto foldSumRec(T arg) {
  return arg;
}

template<typename T, typename... Ts>
auto foldSumRec(T arg1, Ts... otherArgs) {
  return arg1 + foldSumRec(otherArgs...);
}

/// c++17现在的写法
/// 注意括号不能省略
template <typename... T>
auto foldSum(T... args) {
  /// 展开成
  /// args1 + args2 + args3...
  return (... + args);
  /// 展开成 ... arg3 + arg2 + arg1;
  ///return (args + ...);
}

/// 以下是print函数的进化

/// 1> 不会输出空格的版本
template<typename... T>
void print1(T... args) {
  (std::cout << ... << args) << std::endl;
  /// 会展开
  /// std::cout << arg1 << arg2 << arg3 ...
}

template<typename T, typename... Ts>
void print2(const T& first, const Ts&... args) {
  std::cout << first;
  const auto& spaceBefore = [](const auto& arg) {
    std::cout << ' ';
    return arg;
  };
  /// 展开
  /// std::cout << spaceBefore(arg1) << spaceBefore(arg2) << ...
  (std::cout << ... << spaceBefore(args)) << std::endl;
}

template<typename T, typename... Ts>
void print3(const T& first, const Ts&... args) {
  std::cout << first;
  auto outWithSpace = [](const auto &arg) { std::cout << ' ' << arg; };
  /// 展开
  /// outWithSpace(arg1), outWithSpace(arg2) ...
  (..., outWithSpace(args));
  std::cout << '\n';
}
```



7> 扩展的using声明

```c++
/// 也是由于可变模块类型而引出的

/// 继承所有基类里的函数调用运算符
template<typename... Ts>
struct overload : Ts...
{
    /// 将基类中的所有operator()导出到当前的作用域
    using Ts::operator()...;
};

/// 基类的类型从传入的参数中推导
template<typename... Ts>
overload(Ts...) -> overload<Ts...>;

/// 可以如下使用
/// 同时定义两个lambda
auto twice = overload {
                 [](std::string& s) { s += s; },
                 [](auto& v) { v *= 2; }
             };
```



8> 新增标准库optional

```c++
/// optional附带了一个bool变量，用来表示是否有值

/// 该函数可能转化成int，或者为nullopt
  std::optional<int> asInt(const std::string& s) {
    try {
      return std::stoi(s);
    }catch (...) {
      return std::nullopt;
    }
  }

/// optional的构造和取值

/// 没有值的构造
std::optional o1; /// 无值

/// 单一值构造
std::optional o3{42}; /// 推导出std::optional<int>
std::optional o4{"hello"}; /// 推导出std::optional<const char*>

using namespace std::string_literals;
std::optional o5{"hello"s}; /// 推导出std::optional<std::string>

/// 多个参数构造。 使用构造好的对象或者std::in_place作为第一个参数
/// 1> 构造好的对象
std::optional o6{std::complex{3.0, 4.0}};
/// 2> std::in_place作为第一个参数，避免了创建临时对象
std::optional<std::complex<double>> o7{std::in_place, 3.0, 4.0};

/// 通过value()来获取optional的值
if (o14.has_value()) {
  std::cout << o14.value() << std::endl;
}
```



9> 新增标准库variant

```c++
/// variant是更现代版本的union，非常强大!!!

/// 默认会调用第一个类型的默认构造函数
/// 如果第一个类型没有默认够着函数会error
std::variant<std::string, int> var;

/// 可以使用std::monostate来占位第一个类型
/// 保证variant能创建成功
std::variant<std::monostate, int> v2;

/// 通过index()能够访问值所在的索引位置
std::cout << v2.index() << std::endl;

/// 多个参数的构造，需使用in_place_type和in_place_index来标记
variant<complex<double>> v11{std::in_place_type<std::complex<double>>, 3.0, 4.0};
variant<complex<double>> v12{std::in_place_index<0>,3.0,4.0};

/// 访问variant的值!!!

/// 1. get<> get_if<>
std::variant<int, int, std::string> var;
auto a = std::get<0>(var);
if (auto ip = std::get_if<0>(&var); ip != nullptr) {
  std::cout << "有值..." << std::endl;
}

/// 2. 通过visit来访问
/// visitor 必须要有能匹配 variant的函数
struct MyVisitor
{
  void operator()(int& i) const {
    i = 10;
    std::cout << "int: " << i << std::endl;
  }
  void operator()(std::string s) const {
    std::cout << "string: " << s << std::endl;
  }
  void operator()(long double d) const {
    std::cout << "double: " << d << std::endl;
  }
};
std::visit(MyVisitor(), var1);

/// 2.1 通过overload来访问
template<typename... Ts>
struct overload : Ts...
{
  using Ts::operator()...;
};
template<typename... Ts>
overload(Ts...) ->overload<Ts...>;

std::visit(overload{
        [](double i) {std::cout << i << std::endl;},
        [](std::string &s) {std::cout << s << std::endl;}
    }, var1);

/// 2.2 通过泛型lambda来访问
std::visit([](auto &a){
      std::cout << a << std::endl;
    }, var1);

/// !!!! variant可以实现多态
    
/// 声明一个包含所有类型的类型
using Var = std::variant<int, double, std::string>;

std::vector<Var> values{42, 0.19, "study modern cpp", 0.815};
for (const Var& val : values) {
  std::visit([](const auto& v) {
    /// 针对不同类型，做不同的处理
  }, val);
}
```



10> 新增标准库std::any

```c++
/// any 可以存储任意的一个类型

/// 无值构造
std::any a;
/// 一个值的构造
std::any a1{43};
/// 多个值的构造
std::any a2{std::in_place_type<std::complex<double>>, 3, 4};
std::any a3{std::complex<double>{3,4}};
std::any a4 = std::make_any<float>(3.14);

/// 重新赋值
a = 43;
a = "study"; // type is const char *
using namespace std::string_literals;
a = "study"s; // type is string
a.emplace<std::string>("modern cpp");

/// 清空值
a.reset();
a = 43;
a = std::any{};

/// 判断是否有值
std::cout << a.has_value() << std::endl;
/// 判断类型
if (a.has_value()) {
  if (a.type() == typeid(int)) {}
  else if (a.type() == typeid(std::string)) {}
}

/// 取值
a1 = "hello"s;
/// 产生异常版本
auto c = std::any_cast<std::string>(a1);
auto& c1 = std::any_cast<std::string&>(a1);
const auto& c2 = std::any_cast<const std::string&>(a1);
/// 无异常版本, 传递地址过去，转换失败会返回nullptr
if (auto sp = std::any_cast<std::string>(&a1); sp != nullptr) {
}
```



11> 新增标准库string_view

```c++
/// 有一类场景: 
/// 比如从文件中已经读取文本内容到内存了,后续只需要扫描文本内容读取一个个单词
/// 此时是不需要再进行内存的分配，只需要对内存进行索引
/// string_view 就是为这类场景而生!!!!

/// 场景，使用string_view来管理以及分配好的string内存
std::ifstream fin("Variant_Visit.h", std::ios::in);
if (fin.is_open()) {
  std::ostringstream temp;
  temp << fin.rdbuf();
  mTest = temp.str();
  fin.close();
}

/// 此时mTest已经持有的所有的文本内容了，不需要在进行分配内存
/// 用string_view不断的去查看这段文本内容
std::string_view sv{mTest.data()};
/// 不会真正的产生子串对象的内存分配
std::cout << sv.substr(2,10) << std::endl;


/// 使用
const char *c_str_pointer = "study modern cpp,study modern cpp";
char c_str_arr[] = "study modern cpp17,study modern cpp17,study";
std::string cpp_str = "study modern cpp17,study modern cpp17,study";

/// 用c_style pointer初始化，可以指定长度
std::string_view sv1{c_str_pointer};
std::string_view sv2{c_str_pointer, 10};
std::string_view sv3{c_str_arr};
/// 用cpp_str初始化
std::string_view sv4{cpp_str};

/// 取数
for (auto i : sv4) {
  std::cout << i << ' ';
}
std::cout << '\n';

for (auto it = sv4.begin(); it != sv4.end(); ++it) {
  std::cout << *it << ' ';
}
std::cout << '\n';

/// 取出首字符
std::cout << sv4.front() << std::endl;
/// 取出尾字符
std::cout << sv4.back() << std::endl;
/// 获取size
std::cout << sv4.size() << std::endl;
/// 获取指定位置
std::cout << sv4[10] << std::endl;
/// 判断是否为空
std::cout << sv4.empty() << std::endl;
/// 获取字符串内容; 注意 sv4不是以'\0'结尾，一定要以实际的size为主
std::cout << sv4.data() << std::endl;

/// 更改视图大小
sv4.remove_prefix(2); /// 前进2
sv4.remove_suffix(2); /// 结尾缩短2
std::cout << sv4.size() << std::endl;

/// string_view的生命周期是和引用的string相关
/// 引用的string被释放，则string_view不能使用了
```



12> 新增标准库filesystem

```c++
/// 所谓文件系统，就是管理目录和文件的
/// 提供跨平台的技术
/// 判断文件类型，遍历目录，创建目录，方便获取路径的某一个小的分割路径

namespace fs = std::filesystem;

/// 构造一个path
fs::path p{"/xxx/work/modern_cpp/c++17"};

/// 判断路径是否存在
std::cout << fs::exists(p) << "\n";

/// 判断path的类型， 普通文件还是一个目录?
switch (status(p).type()) {
    /// 非隐藏文件
  case fs::file_type::regular: {
    auto err = std::error_code{};
    auto fileSize = fs::file_size(p, err);
    /// 是普通文件可以获取文件大小/文件名
    std::cout << fileSize << ", " << p.filename() << std::endl;
    break;
  }
  case fs::file_type::directory: {
    /// 如果是目录，可以方便的遍历目录下的文件
    for (auto &entry : fs::directory_iterator(p)) {
      if (entry.status().type() == fs::file_type::regular)
        std::cout << entry.path() << std::endl;
      else if (entry.status().type() == fs::file_type::directory) {
        /// 可以继续遍历, 可以实现成一个递归遍历
      }
    }
    break;
  }
  default:
    break;
}
```

