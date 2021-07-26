# C++ 算法练习常用 API 总结

该文总结了 C++ 刷题 **常用 API**。

目录：

- [C++ 算法练习常用 API 总结](#c-算法练习常用-api-总结)
  - [数组](#数组)
    - [静态数组](#静态数组)
      - [声明及初始化](#声明及初始化)
      - [遍历数组](#遍历数组)
        - [一维数组](#一维数组)
        - [二维数组](#二维数组)
      - [数组排序](#数组排序)
    - [动态数组 - vector](#动态数组---vector)
      - [声明及初始化](#声明及初始化-1)
      - [遍历数组](#遍历数组-1)
        - [一维数组](#一维数组-1)
        - [二维数组](#二维数组-1)
      - [数组排序](#数组排序-1)
      - [常用函数](#常用函数)
  - [字符串](#字符串)
    - [声明及初始化](#声明及初始化-2)
    - [遍历字符串](#遍历字符串)
    - [常用函数](#常用函数-1)
  - [队列](#队列)
  - [堆栈](#堆栈)
  - [集合](#集合)
  - [哈希](#哈希)
  - [参考链接](#参考链接)

## 数组

### 静态数组

#### 声明及初始化

声明数组：

```C++
type arrayName[arraySize];
```

在 C++ 中，可以逐个初始化数组元素，也可以使用一个初始化语句，如下所示：

```C++
// 大括号 { } 之间的值的数目不能大于在数组声明时在方括号 [ ] 中指定的元素数目。
double balance[5] = {1000.0, 2.0, 3.4, 7.0, 50.0}; 

// 如果省略掉了数组的大小，数组的大小则为初始化时元素的个数。
double balance[] = {1000.0, 2.0, 3.4, 7.0, 50.0}; 
```

#### 遍历数组

##### 一维数组

**eg 1.1 下标 for 循环遍历**

```C++
int a[5] = {0, 3, 4, 6, 2};
int size = sizeof(a) / sizeof(*a);
for(int i = 0; i < size; i++)
    cout << a[i] << " ";
```

**eg 1.2 基于范围的 for 循环遍历**

```C++
for(int &item : a)
    cout << item << " ";
```

**eg 1.3 基于指针的 for 循环遍历**

```C++
// 此处 auto p 等价于 int *p
for (auto p = a; p != a + size; p++) {
  cout << *p;
}
```

##### 二维数组

**eg 2.1 下标 for 循环遍历**

```C++
int a[3][4] = {1,2,3,4,5,6,7,8};
for (int i = 0; i < 3; i++) {
  for (int j = 0; j < 4; j++) {
    cout << a[i][j] << endl;
  }
}
```

**eg 2.2 基于范围的 for 循环遍历**

```C++
for (auto &p : a) {
  for (auto &q : p) {
    cout << q << endl;
  }
}
```

> **注意：** 上述示例 `auto p` 被转化为指针（其类型为 `int(*)[4]`），而在内循环中指针是不可以范围 for 循环遍历的，所以会报错。故使用基于范围的 for 循环遍历时，除最里层循环以外的所有循环都必须用引用，才能正常遍历。并且如果还想对元素赋值，最里层循环也需使用引用。

**eg 2.3 基于指针的遍历**

```C++
int ia[3][4] = {1,2,3,4,5,6,7,8};

// 此处 auto p 等价于 int (*p)[4]
for (auto p = ia; p != ia + 3; p++) {
  for (auto q = *p; q != *p + 4; q++) {
    cout << *q << endl;
  }
}

for (auto p = begin(ia); p != end(ia); p++) {
  for (auto q = begin(*p); q != end(*p); q++) {
    cout << *q << endl;
  }
}
```

> **注意：** 多维数组，除最外层循环外，其余循环均需要对指针进行解引用。

#### 数组排序

```C++
// 参数 1 - 待排序数组的首地址，参数 2 - 待排序数组的尾地址，参数 3 - 排序方法，默认升序
sort(array, array + size);
```

其中，`sort()` 方法包含在 `<algorithm>` 头文件中，并定义在 `std` 命令空间：

```C++
#include <algorithm>
using namespace std;
```

### 动态数组 - vector

#### 声明及初始化

使用 `vector` 容器时，需引用头文件 `<vector>`。

```C++
#include <vector>

/* eg 1 initialize */

// 初始化一个 int 类型的空数组 nums
vector<int> nums;

// 初始化一个大小为 n 的数组 nums，数组中的值默认都为 0
vector<int> nums(n);

// 初始化一个元素为 1、3、5 的数组 nums
vector<int> nums{1, 3, 5};

// 初始化一个大小为 n 的数组 nums，其值全都为 2
vector<int> nums(n, 2);

// 初始化一个二维 int 数组 dp
vector<vector<int>> dp;

// 初始化一个大小为 m*n 的布尔数组 dp，其值均为 true
vector<vector<bool>> dp(m, vector<bool>(n, true));

/* eg 2 make a copy */
vector<int> v(v1.begin(), v1.end());
vector<int> v(v2);

/* eg 3 cast an array to a vector */
int a[5] = {0, 1, 2, 3, 4};
vector<int> v(a, *(&a + 1));
```

#### 遍历数组

##### 一维数组

**eg 1.1 下标 for 循环遍历**

```C++
for (int i = 0; i < v1.size(); ++i) {
  cout << v1[i] << endl;
}
```

**eg 1.2 基于范围的 for 循环遍历**

```C++
for (int &item : v2) {
  cout << item << endl;
}
```

**eg 1.3 基于指针的 for 循环遍历**

```C++
for (auto item = v3.begin(); item != v4.end(); ++item) {
  cout << *item << endl;
}
```

##### 二维数组

**eg 2.1 下标 for 循环遍历**

```C++
for (int i = 0; i < vec.size(); i++) {
  for(int j = 0; j < vec[0].size(); j++) {
    cout << vec[i][j] << " ";
  }
}
```

**eg 2.2 基于范围的 for 循环遍历**

```C++
for (auto &p : vec) {
  for (auto &q : p) {
    cout << q << "  ";
  }
}
```

**eg 2.3 基于指针的 for 循环遍历**

```C++
for (auto p = vec.begin(); p != vec.end(); p++) {
  for (auto q = (*p).begin(); q != (*p).end(); q++) {
    cout << *q << "  ";
  }
}
```

**eg 2.4 基于迭代器遍历**

#### 数组排序

```C++
sort(v.begin(), v.end());
```

#### 常用函数

```C++
// 返回数组是否为空
bool empty();

// 返回数组的元素个数
size_type size();

// 返回数组最后一个元素的引用
reference back();

// 在数组尾部插入一个元素 val
void push_back(const value_type &val);

// 删除数组尾部的那个元素
void pop_back();
```

## 字符串

### 声明及初始化

使用 `string` 类时，需引用头文件 `<string>`。

```C++
#include <iostream>
#include <string>

string s1;
string s2 = "abc";
```

### 遍历字符串

TODO

### 常用函数

```C++
// 返回字符串的长度
size_t size();

// 判断字符串是否为空
bool empty();

// 在字符串尾部插入一个字符 c
void push_back(char c);

// 删除字符串尾部的字符
void pop_back();

// 返回从索引 pos 开始，长度为 len 的子字符串
string substr(size_t pos, siez_t len);

// 查找子串（返回 str2 在 str1 中的下标位置，如果没有找到，则返回 npos（int 型））
str1.find(str2);

// string 连接
s1 = s1 + s2;
s1.append(s2);

// string 比较（相等时返回 0；s1 > s2 返回 1，s1 < s2 返回 -1）
if (s1 > s2) ......
s1.compare(s2);
```

## 队列

## 堆栈

## 集合

## 哈希

TODO

## 参考链接

* [C++ 数组](https://www.runoob.com/cplusplus/cpp-arrays.html)
* [C++ 数组和vector的基本操作](https://www.cnblogs.com/HL-space/p/10546585.html)
* [C++多维数组的遍历以及初始化](https://blog.csdn.net/anlian523/article/details/90549379)
* [C++ string类（C++字符串）完全攻略](http://c.biancheng.net/view/400.html)