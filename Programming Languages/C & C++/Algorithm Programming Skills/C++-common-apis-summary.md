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
      - [基本操作](#基本操作)
  - [字符串](#字符串)
    - [声明及初始化](#声明及初始化-2)
    - [遍历字符串](#遍历字符串)
    - [基本操作](#基本操作-1)
    - [类型转换](#类型转换)
      - [数值 转换为 string](#数值-转换为-string)
      - [string 转换为 数值](#string-转换为-数值)
  - [队列](#队列)
    - [声明及初始化](#声明及初始化-3)
    - [基本操作](#基本操作-2)
  - [堆栈](#堆栈)
    - [声明及初始化](#声明及初始化-4)
    - [基本操作](#基本操作-3)
  - [集合](#集合)
    - [声明及初始化](#声明及初始化-5)
    - [遍历集合](#遍历集合)
    - [基本操作](#基本操作-4)
  - [哈希表](#哈希表)
    - [声明及初始化](#声明及初始化-6)
    - [遍历哈希表](#遍历哈希表)
    - [基本操作](#基本操作-5)
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
for(int i = 0; i < size; ++i)
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
for (auto p = a; p != a + size; ++p) {
  cout << *p;
}
```

##### 二维数组

**eg 2.1 下标 for 循环遍历**

```C++
int a[3][4] = {1,2,3,4,5,6,7,8};
for (int i = 0; i < 3; ++i) {
  for (int j = 0; j < 4; ++j) {
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
for (auto p = ia; p != ia + 3; ++p) {
  for (auto q = *p; q != *p + 4; ++q) {
    cout << *q << endl;
  }
}

for (auto p = begin(ia); p != end(ia); ++p) {
  for (auto q = begin(*p); q != end(*p); ++q) {
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

其中，`sort()` 方法包含在 `<algorithm>` 头文件中，并定义在 `std` 命名空间：

```C++
#include <algorithm>
using namespace std;
```

### 动态数组 - vector

#### 声明及初始化

使用 `vector` 容器时，需引入头文件 `<vector>`。

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

**eg 1.3 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto i = vec.begin()
for (vector<int>::iterator i = vec.begin(); i != vec.end(); ++i) {
    cout << *i << "  ";
}
```

迭代器按照定义方式分为以下四种：

* 正向迭代器

  ```C++
  容器类名::iterator  迭代器名;
  ```

* 常量正向迭代器

  ```C++
  容器类名::const_iterator  迭代器名;
  ```

* 反向迭代器

  ```C++
  容器类名::reverse_iterator  迭代器名;
  ```

* 常量反向迭代器

  ```C++
  容器类名::const_reverse_iterator  迭代器名;
  ```

##### 二维数组

**eg 2.1 下标 for 循环遍历**

```C++
for (int i = 0; i < vec.size(); ++i) {
  for(int j = 0; j < vec[0].size(); ++j) {
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

**eg 2.3 基于迭代器遍历**

```C++
for (vector<vector<int>>::iterator i = vec.begin(); i != vec.end(); ++i) {
  for (vector<int>::iterator j = (*i).begin(); j != (*i).end(); ++j) {
      cout << *j << " ";
  }
  cout << endl;
}

// 可使用 auto 关键字简化
for (auto p = vec.begin(); p != vec.end(); ++p) {
  for (auto q = (*p).begin(); q != (*p).end(); ++q) {
      cout << *q << "  ";
  }
}
```

#### 数组排序

```C++
sort(v.begin(), v.end());
```

#### 基本操作

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

**eg 1.1 下标 for 循环遍历**

```C++
for (int i = 0; i < a.size(); ++i) {
    cout << a[i] << " ";
}
```

**eg 1.2 基于范围的 for 循环遍历**

```C++
for (char c : a) {
    cout << c << "  ";
}
```

**eg 1.3 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto i = a.begin();
for (string::iterator i = a.begin(); i != a.end(); ++i) {
    cout << *i << " ";
}
```

### 基本操作

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

### 类型转换

使用以下转化方法时，需引入头文件 `<string>`，并使用 `std` 命名空间。

#### 数值 转换为 string

* `to_string`  : numerical value to string
* `to_wstring` : numerical value to wide string

#### string 转换为 数值

* `stoi`   : string to integer
* `stol`   : string to long int
* `stoul`  : string to unsigned integer
* `stoll`  : string to long long
* `stoull` : string to unsigned long long
* `stof`   : string to float
* `stod`   : string to double
* `stold`  : string to long double

## 队列

### 声明及初始化

使用队列 `queue` 时，需引入头文件 `<queue>`。

```C++
#include <queue>

queue<int> q;
```

### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type  size();

// 将元素加入队尾
void push(const value_type& val);

// 返回队头元素的引用
value_type& front();

// 删除队头元素
void pop();
```

> **注意：** `pop()` 方法为 `void` 类型，不会返回被删除的元素，所以元素出队的一般做法为：
> `int val = q.front(); q.pop();`

## 堆栈

### 声明及初始化

使用堆栈 `stack` 时，需引入头文件 `<stack>`。

```C++
#include <stack>

stack<int> s;
```

### 基本操作

```C++
// 返回堆栈是否为空
bool empty();

// 返回堆栈中元素的个数
size_type size();

// 在栈顶添加元素
void push(const value_type& val);

// 返回栈顶元素的引用
value_type& top();

// 删除栈顶元素
void pop();
```

## 集合

### 声明及初始化

使用集合 `unordered_set` 时，需引入头文件 `<unordered_set>`。

```C++
#include <unordered_set>

unordered_set<int> set;
```

### 遍历集合

**eg 1. 基于范围的 for 循环遍历**

```C++
for (auto val : set) {
    cout << val << endl;
}
```

**eg 2. 基于迭代器遍历**

通过迭代器 `iterator` 遍历。

```C++
unordered_set<int> set;

// 可用 auto 关键字简化 auto iter = set.begin()
for (unordered_set<int>::iterator iter = set.begin(); iter != set.end(); ++iter) {
    cout << *iter << " ";
}
```

### 基本操作

```C++
// 返回集合中的元素个数
size_type size();

// 返回集合是否为空，如果为空返回 1，否则返回 0
bool empty();

// 如果 key 存在返回 1，否则返回 0
size_type count();

// 向集合中插入一个元素 key
pair<iterator, bool> insert(const key_type& key);

// 删除集合中的元素 key，如果删除成功则返回 1，否则返回 0
size_type erase(const key_type& key);
```

## 哈希表

### 声明及初始化

使用哈希表 `unordered_map` 时，需引入头文件 `<unordered_map>`。

```C++
#include <unordered_map>

unordered_map<int, int> mapping;
```

### 遍历哈希表

**eg 1. 下标 for 循环遍历**

```C++
for (int i = 0; i < mapping.size(); ++i) {
    cout << i << " " << mapping[i] << endl;
}
```

**eg 2. 基于范围的 for 循环遍历**

```C++
for (auto item : mapping) {
    cout << item.first << " " << item.second << endl;
}
```

**eg 3. 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto iter = mapping.begin()
for (unordered_map<int, string>::iterator iter = mapping.begin(); iter != mapping.end(); ++iter) {
    cout << iter->first << endl;
    cout << iter->second << endl;
}
```

### 基本操作

```C++
// 返回哈希表中的键值对个数
size_type size();

// 返回哈希表是否为空，如果不为空返回 1，否则返回 0
bool empty();

// 如果 key 存在返回 1，否则返回 0
size_type count(const key_type& key);

// 通过 key 删除哈希表中的键值对，如果删除成功则返回 1，否则返回 0
size_type erase(const key_type& key);
```

> **注意：** 用 `[]` 访问其中的键 `key` 时，如果 `key` 不存在，则会自动创建 `key`，对应的值为该值类型的默认值。

## 参考链接

* [C++ 数组](https://www.runoob.com/cplusplus/cpp-arrays.html)
* [C++ 数组和vector的基本操作](https://www.cnblogs.com/HL-space/p/10546585.html)
* [C++多维数组的遍历以及初始化](https://blog.csdn.net/anlian523/article/details/90549379)
* [C++迭代器（STL迭代器）iterator详解](http://c.biancheng.net/view/338.html)
* [C++ string类（C++字符串）完全攻略](http://c.biancheng.net/view/400.html)
* [C++ string Reference](http://www.cplusplus.com/reference/string/)
