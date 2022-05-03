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
    - [partial_sum / adjacent_difference](#partial_sum--adjacent_difference)
  - [搜索](#搜索)
    - [二分搜索](#二分搜索)
      - [binary_search](#binary_search)
      - [lower_bound / upper_bound](#lower_bound--upper_bound)
  - [赋值](#赋值)
    - [iota](#iota)
  - [参考链接](#参考链接)

## 极值

### max / min

`std::max` / `std::min` 函数定义在头文件 `<algorithm>` 中，C++ 11 标准下的函数原型如下（min 函数与 max 类似）：

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

`std::max_element` / `std::min_element` 函数定义在头文件 `<algorithm>` 中，其函数原型如下（min_element 函数与 max_element 类似）：

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

`std::accumulate` 函数定义在头文件 `<numeric>` 中，其函数原型如下：

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

`std::count` / `std::count_if` 函数定义在头文件 `<algorithm>` 中，两者函数原型如下：

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

### partial_sum / adjacent_difference

`std::partial_sum` / `std::adjacent_difference` 函数定义在头文件 `<numeric>` 中，其函数原型如下（adjacent_difference 类似）：

```C++
sum (1)	
template <class InputIterator, class OutputIterator>
   OutputIterator partial_sum (InputIterator first, InputIterator last,
                               OutputIterator result);
custom (2)	
template <class InputIterator, class OutputIterator, class BinaryOperation>
   OutputIterator partial_sum (InputIterator first, InputIterator last,
                               OutputIterator result, BinaryOperation binary_op);
```

* `partial_sum` 用于计算 `[first,last)` 范围内的元素的前缀和。默认为加法，可以指定不同的操作 `binary_op`。
* `adjacent_difference` 用于计算 `[first,last)` 范围内的元素的差分。默认为计算差值，可以指定不同的操作 `binary_op`。

常见用法：

```C++
vector<int> arr = {5, 3, 5, 8, 16}; // 原始数组
int res[5];                         // 结果保存数

/* partial_sum() */

// 默认加法 - 前缀和
partial_sum(arr.begin(), arr.end(), res);
for (auto &val : res) cout << val << " "; // 5 8 13 21 37

// 自定义运算符 - 前缀乘
partial_sum(arr.begin(), arr.end(), res, multiplies<>());
for (auto &val : res) cout << val << " "; // 5 15 75 600 9600

/* adjacent_difference() */

// 默认减法 - 差分
adjacent_difference(arr.begin(), arr.end(), res);
for (auto &val : res) cout << val << " "; // 5 -2 2 3 8
```

## 搜索

### 二分搜索

#### binary_search

`std::binary_search` 函数定义在头文件 `<algorithm>` 中，其函数原型如下：

```C++
default (1)	
template <class ForwardIterator, class T>
  bool binary_search (ForwardIterator first, ForwardIterator last,
                      const T& val);
custom (2)	
template <class ForwardIterator, class T, class Compare>
  bool binary_search (ForwardIterator first, ForwardIterator last,
                      const T& val, Compare comp);
```

如果 `[first,last)` 范围内的任何元素等价于 `val`，则返回 `true`，否则返回 `false`。

常见用法：

```C++
// default (1) - 升序
vector<int> arr = {1, 3, 2, 8, 5, 6};
sort(arr.begin(), arr.end());
if (binary_search(arr.begin(), arr.end(), 2)) cout << "Found" << endl;

// default (1) - 降序
sort(arr.begin(), arr.end(), greater<int>());
if (binary_search(arr.begin(), arr.end(), 2, greater<int>())) cout << "Found" << endl;
```

#### lower_bound / upper_bound

`std::lower_bound` / `std::upper_bound` 函数定义在头文件 `<algorithm>` 中，其函数原型如下（`upper_bound` 函数与 `lower_bound` 类似）：

```C++
default (1)	
template <class ForwardIterator, class T>
  ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,
                               const T& val);
custom (2)	
template <class ForwardIterator, class T, class Compare>
  ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,
                               const T& val, Compare comp);
```

* `lower_bound` 返回一个迭代器，指向范围 `[first,last)` 中 `>= val` 的第一个元素。
* `upper_bound` 返回一个迭代器，指向范围 `[first,last)` 中 `> val` 的第一个元素。

常见用法：

```C++
vector<int> arr = {1, 3, 5, 8, 7, 6};
auto it_lower = lower_bound(arr.begin(), arr.end(), 5);
auto it_upper = upper_bound(arr.begin(), arr.end(), 5);

// 输出下标和值
cout << it_lower - arr.begin() << " " << *it_lower << endl; // 2 5
cout << it_upper - arr.begin() << " " << *it_upper << endl; // 3 8
```

## 赋值

### iota

`std::iota` 函数定义在头文件 `<numeric>` 中，其函数原型如下：

```C++
template <class ForwardIterator, class T>
  void iota (ForwardIterator first, ForwardIterator last, T val);
```

主要用于给 `[first,last)` 范围内的元素，依次累加赋值，等价于 `arr[idx++] = val++`。

常见用法：

```C++
// 初始化数组元素值等于其下标值
vector<int> arr(10);
iota(arr.begin(), arr.end(), 0);
for (auto &val : arr) cout << val << " "; // 输出：0 1 2 3 4 5 6 7 8 9
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [高效位运算 __builtin_系列函数
](https://blog.csdn.net/yuer158462008/article/details/46383635?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_paycolumn_v3&spm=1001.2101.3001.4242.1&utm_relevant_index=2)
