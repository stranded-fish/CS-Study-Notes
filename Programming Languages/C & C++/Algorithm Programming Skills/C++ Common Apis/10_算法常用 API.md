# C++ 算法常用 API

目录：

- [C++ 算法常用 API](#c-算法常用-api)
  - [极值](#极值)
    - [max / min](#max--min)
    - [max_element / min_element](#max_element--min_element)
  - [统计](#统计)
    - [accumulate](#accumulate)
    - [count / count_if](#count--count_if)
    - [__builtin](#__builtin)
  - [搜索](#搜索)
    - [upper_bound() lower_bound()](#upper_bound-lower_bound)
  - [参考链接](#参考链接)

## 极值

### max / min

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

### max_element / min_element

`max_element` / `min_element` 函数定义在头文件 `<algorithm>` 中。

函数原型如下（min_element 函数与 max_element 类似）：

```C++
default (1)	
template <class ForwardIterator>
  ForwardIterator max_element (ForwardIterator first, ForwardIterator last);
custom (2)	
template <class ForwardIterator, class Compare>
  ForwardIterator max_element (ForwardIterator first, ForwardIterator last,
                               Compare comp);
```

该方法返回一个迭代器，该迭代器指向 `[first,last)` 范围内具有最大（小）值的元素。如果有多个最大（小）值，则会指向所遇到的第一个值。

常见用法：

```C++
vector<int> arr = {1, 3, 4, 5, 6, 7, 8, 2};
cout << *min_element(arr.begin(), arr.end()); // 1
cout << *max_element(arr.begin(), arr.end()); // 8
```

## 统计

### accumulate

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

### count / count_if

`count` / `count_if` 函数定义在头文件 `<algorithm>` 中。

两者函数原型如下：

```C++
/* count - 返回范围 [first,last) 中比较等于 val 的元素数量。
该函数使用 operator== 将各个元素与 val 进行比较。*/
template <class InputIterator, class T>
  typename iterator_traits<InputIterator>::difference_type
    count (InputIterator first, InputIterator last, const T& val);

// count_if - 返回范围 [first,last) 中 pred 为 true 的元素数量。
template <class InputIterator, class UnaryPredicate>
  typename iterator_traits<InputIterator>::difference_type
    count_if (InputIterator first, InputIterator last, UnaryPredicate pred);
```

常见用法：

```C++
vector<int> arr = {1, 3, 4, 5, 6, 7, 8, 2, 2, 2, 2, 3};
cout << count(arr.begin(), arr.end(), 2); // 输出元素 2 的数量
cout << count_if(arr.begin(), arr.end(),  // 输出奇数元素的数量
        [](const int &val){ return val % 2; });
```

### __builtin

* `int __builtin_popcount (unsigned int x)`
  * 返回二进制中 `1` 的个数。
* `int __builtin_clz (unsigned int x)`
  * 返回前导 `0` 的个数。
* `int __builtin_ctz (unsigned int x)`
  * 返回后导 `0` 的个数，和 `__builtin_clz` 相对。
* `int __builtin_ffs (unsigned int x)`
  * 返回 `x` 的最后一位 `1` 的倒序位置，比如 7368（1110011001000）返回 4。

以上函数均有相应的 `usigned long` 和 `usigned long long` 版本，只需要在函数名后面加上 `l` 或 `ll`，如 `int __builtin_clzll`。

## 搜索

### upper_bound() lower_bound()


## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [高效位运算 __builtin_系列函数
](https://blog.csdn.net/yuer158462008/article/details/46383635?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_paycolumn_v3&spm=1001.2101.3001.4242.1&utm_relevant_index=2)