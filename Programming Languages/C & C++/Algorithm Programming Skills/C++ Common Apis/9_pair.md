# C++ pair

目录：

- [C++ pair](#c-pair)
  - [声明及初始化](#声明及初始化)
  - [基本操作](#基本操作)
  - [参考链接](#参考链接)

## 声明及初始化

使用 `pair` 类型时，需引入头文件 `<utility>`。

类模板：`template<class T1,class T2> struct pair`

* `T1` - 第一个值的数据类型
* `T2` - 第二个值的数据类型

```C++
#include <utility>

// 初始化一个 pair 对象，其成员值为该数据类型的默认值
pair<T1, T2> p1;

// 初始化一个 pair 对象，first = v1, second = v2
pair<T1, T2> p1(v1, v2);

 // 调用拷贝构造函数
pair<T1, T2> p1(p2);
```

## 基本操作

```C++
// 创建对象
pair<T1, T2> make_pair(v1, v2);

// 访问成员变量
p.first  // v1
p.second // v2

/* 运算符操作
pair 对象重载了 <、<=、>、>=、==、!= 运算符，其运算规则是：
对于进行比较的 2 个 pair 对象，先比较 pair.first 元素的大小，如果相等则继续比较 pair.second 元素的大小。
注意：参与运算的 pair 对象的数据类型必须相同 */
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [C++ STL pair用法详解](http://c.biancheng.net/view/7169.html)
