# C++ 算法常用 API

目录：

- [C++ 算法常用 API](#c-算法常用-api)
  - [max / min](#max--min)
  - [accumulate](#accumulate)
  - [参考链接](#参考链接)

## max / min

`max` / `min` 函数定义在头文件 `<algorithm>` 中。

C++ 11 标准下的函数原型如下（min 函数与 max 类似）：

```C++
default (1) 
template <class T> const T& max (const T& a, const T& b);

custom  (2) 
template <class T, class Compare>
    const T& max (const T& a, const T& b, Compare comp);

initializer list (3) 
template <class T> T max (initializer_list<T> il);
template <class T, class Compare>
    T max (initializer_list<T> il, Compare comp);
```

常见用法：

```C++
int a = 1, b = -2, c = 3;

// default (1)
max(a,b); // a: 1

// custom (2)
bool comp(const int a, const int b) { return a < b * b; }
max(a, b, comp) // b: -2

// initializer list (3) 
max({a, b, c}); // c: 3
```

## accumulate

`accumulate` 函数定义在头文件 `<numeric>` 中。

函数原型如下：

```C++
sum (1)	
template <class InputIterator, class T>
    T accumulate (InputIterator first, InputIterator last, T init);

custom (2)	
template <class InputIterator, class T, class BinaryOperation>
    T accumulate (InputIterator first, InputIterator last, T init,
                 BinaryOperation binary_op);
```

该方法 sum (1) 接收三个参数：

* 前两个参数指出了需要求和的元素的范围；
* 第三个参数为和的初值。
  * 该参数同时决定了函数中使用哪个加法运算符以及返回值类型，如果前两个参数所指的类型没有该加法运算符，则调用时会产生编译错误。

常见用法：

```C++
vector<int> arr = {1,2,3,4};

// 对 arr 中的元素求和，和的初值为 0
int sum = accumulate(arr.begin(), arr.end(), 0); // sum = 10
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
