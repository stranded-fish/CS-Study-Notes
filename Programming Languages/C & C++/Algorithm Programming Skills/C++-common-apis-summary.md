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
    - [动态数组 - vector](#动态数组---vector)
      - [声明及初始化](#声明及初始化-1)
      - [遍历数组](#遍历数组-1)
        - [一维数组](#一维数组-1)
        - [二维数组](#二维数组-1)
      - [基本操作](#基本操作)
    - [数组排序](#数组排序)
  - [列表](#列表)
    - [声明及初始化](#声明及初始化-2)
    - [遍历列表](#遍历列表)
    - [基本操作](#基本操作-1)
  - [字符串](#字符串)
    - [声明及初始化](#声明及初始化-3)
    - [遍历字符串](#遍历字符串)
    - [基本操作](#基本操作-2)
    - [扩展操作](#扩展操作)
      - [字符串逆转](#字符串逆转)
      - [处理 string 对象中的字符](#处理-string-对象中的字符)
      - [字符串分割](#字符串分割)
    - [类型转换](#类型转换)
      - [数值 转换为 string](#数值-转换为-string)
      - [string 转换为 数值](#string-转换为-数值)
  - [队列](#队列)
    - [普通队列](#普通队列)
      - [声明及初始化](#声明及初始化-4)
      - [基本操作](#基本操作-3)
    - [双端队列](#双端队列)
      - [声明及初始化](#声明及初始化-5)
      - [基本操作](#基本操作-4)
    - [优先队列](#优先队列)
      - [声明及初始化](#声明及初始化-6)
      - [基本操作](#基本操作-5)
      - [自定义排序](#自定义排序)
  - [堆栈](#堆栈)
    - [声明及初始化](#声明及初始化-7)
    - [基本操作](#基本操作-6)
  - [集合](#集合)
    - [声明及初始化](#声明及初始化-8)
    - [遍历集合](#遍历集合)
    - [基本操作](#基本操作-7)
  - [哈希表](#哈希表)
    - [声明及初始化](#声明及初始化-9)
    - [遍历哈希表](#遍历哈希表)
    - [基本操作](#基本操作-8)
  - [pair](#pair)
    - [声明及初始化](#声明及初始化-10)
    - [基本操作](#基本操作-9)
  - [其他操作](#其他操作)
    - [max / min](#max--min)
    - [accumulate](#accumulate)
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

// 数组值初始化为全 0
int array[10] = {};
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

#### 基本操作

**基础操作：**

```C++
// 返回数组是否为空
bool empty();

// 返回数组的元素个数
size_type size();

// 返回数组最后一个元素的引用
reference back();

// 在数组尾部插入一个元素 val
void push_back(const value_type &val);
void emplace_back(Args&&... args);

// 删除数组尾部的那个元素
void pop_back();

// 将数组 size 设置为 0，但 capacity 不变（注意：不能用于清空数组）
void clear(); 

// 交换两个 vector
newArray.swap(oldArray);
```

**数组逆转：**

使用 `reverse()` 函数时，需引入头文件 `<algorithm>`。

```C++
#include <algorithm>
void reverse<_BidIt>(const _BidIt _First, const _BidIt _Last);

vector<int> test{1, 2, 3};

// 逆转整个数组
reverse(test.begin(), test.end()); // test{3, 2, 1}

// 逆转部分数组
reverse(test.begin(), test.begin() + 2); // test{2, 1, 3}
```

**数组拷贝：**

```C++
vector<int> oldArray{1, 2, 3};
vector<int> newArray;

// eg 1 - 赋值运算符 深拷贝
newArray = oldArray;

// eg 2 - assign() 清空并深拷贝
newArray.assign(oldArray.begin(), oldArray.end());
```

**插入元素：**

```C++
vector<int> array{1, 2};

// 插入单个元素
iterator insert(const_iterator position, const value_type& val);
array.insert(array.begin() + 1, 3); // {1,3,2}

// 填充 n 个元素
iterator insert(const_iterator position, size_type n, const value_type& val);
array.insert(array.end(), 2, 5); // {1,2,5,5}

// 插入 [first,last) 范围内的所有元素
iterator insert(const_iterator position, InputIterator first, InputIterator last);
vector<int> test{7,8,9};
array.insert(array.end(), test.begin(), test.end()); // {1,2,7,8,9}
```

**调整容量：**

* `reserver` 调整容器预分配存储区大小，即 capacity 值，但并不进行初始化：

```C++
void reserve(const size_t _Newcapacity)
```

> **注意：** reserve 的参数 n 是推荐预分配内存的大小，实际分配的可能等于或大于这个值，即 n 大于 capacity 的值，就会 reallocate 内存，capacity 的值会大于或者等于 n 。这样，当容器调用 push_back 函数使得 size 超过原来的默认分配的 capacity 值时，就能避免内存重分配开销。
>
> 同时，reserve 函数分配出来的内存空间，只是表示 vector 可以利用这部分内存，但由于该部分内存还未进行初始化，故不能有效地访问这些内存空间，访问的时候会出现越界现象，导致程序崩溃。

* `resize` 调整容器大小，并且创建对象：

```C++
// 调整容器大小，使其包含 n 个元素，如果有新增元素，则将新元素进行默认初始化
void resize (size_type n); 

// 调整容器大小，使其包含 n 个元素，如果有新增元素，则将新元素初始化为 val 的副本
void resize (size_type n, const value_type& val);
```

* 如果 n 小于当前容器的大小，则将内容减少到其前 n 个元素，并删除超出范围的元素（并销毁它们）。
* 如果 n 大于当前容器的大小，则通过在末尾插入所需数量的元素来扩展内容，以达到 n 的大小。如果指定了 val，则将新元素初始化为 val 的副本，否则将对它们进行值初始化。
* 如果 n 也大于当前容器容量，将自动重新分配已分配的存储空间。

> **注意：** 该函数通过插入或擦除容器中的元素来更改容器的实际内容。

### 数组排序

`sort()` 方法能够对容器或普通数组中 `[first, last)` 范围内的元素进行排序，默认进行升序排序。

该方法包含在 `<algorithm>` 头文件，并定义在 `std` 命名空间：

```C++
#include <algorithm>
using namespace std;
```

`sort()` 函数原型如下：

```C++
default (1)	
  void sort (RandomAccessIterator first, RandomAccessIterator last);
custom  (2)	
  void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```

* `first` - 待排序数组的首地址
* `last`  - 待排序数组的尾地址
* `comp`  - 排序方法，缺省为升序

其中 `custom (2)` 自定义排序，可使用 C++ 内置排序方法：

* `less<data-type>()`：升序
* `greater<data-type>()`：降序

也可以使用自定义排序方法：

```C++
bool comp(const Type1 &a, const Type2 &b);

// 示例：升序排序
bool comp(const int &a, const int &b) {
    return a < b;
}

vector<int> nums;
sort(nums.begin(), nums.end(), comp);
```

其中自定义排序方法，比较入参 `a` 和 `b`：

* 升序：定义当 `a < b` 时，返回 `true`；
* 降序：定义当 `a > b` 时，返回 `true`；

> **注意：**
> **①** 出于效率考虑，为了避免多余拷贝，应采用引用传递。同时该函数不得修改其任何参数，故需要用 `const` 限定符加以限定。
>
> **②** 当在类中使用自定义 `comp` 方法进行排序时，不能直接调用类中定义的非静态 `comp` 方法（如：LeetCode 核心代码模式）。
>
> 因为非静态的成员函数必须要被绑定到一个类的对象或者指针上，才能得到被调用对象的 `this` 指针，为了区分是谁调用了成员函数，就必须要有 `this` 指针，`this` 指针是隐式添加到函数参数列表中的，而这就使得与所需 `comp` 方法入参不匹配。
>
> 解决方法：
>
> 1. 通过 `static` 修饰自定义排序方法 `comp`。由于类的静态成员不依赖于具体对象，所有实例化对象都共享同一个静态成员，故也就没有 `this` 指针的概念。
> 2. 将该方法定义到类外部，即定义为全局普通函数。
>
>
> **③** Effective STL no.21：总是让比较函数在等值情况下返回 false。
>
> sort 函数为了最大程度的提高效率，结合了快排、堆排和插入排序等多种排序方法，分为 std::__introsort_loop 和 std::__final_insertion_sort 两个阶段。
>
> 第一阶段使用 “快排+堆排” 的方法，第二阶段使用 “插入排序”，当元素个数小于等于 _S_threshold（enum {_S_threshold = 16 }）时，执行普通的插入排序，当大于 _S_threshold 时，执行两次的 “插入” 排序操作，首先使用普通的插入排序来排 [first,_S_threshold) 这个范围的元素，然后使用无保护的插入排序，完成 [_S_threshold, last) 这个范围的排序。
>
> 而最后使用的无保护的插入排序，为了提升效率，省略了越界的检查。如果在比较元素相等的情况下返回 true 会导致遍历无法停止，最终造成访问越界，程序崩溃。
>
> 解决方法，遵循自定义 [Compare 实现要求](https://en.cppreference.com/w/cpp/named_req/Compare)：
>
> 1. 对于任意元素 a，需满足 comp(a, a) == true；
> 2. 对于任意两个元素 a 和 b，若 comp(a, b) == true 则要满足 comp(b, a) == false；
> 3. 对于任意三个元素 a、b 和 c，若 comp(a, b) == true 且 comp(b, c) == true 则需要满足 comp(a, c) == true。
>
> 综上：如果定义两个元素相等时返回 true，将无法满足条件 2，故总是需要让比较函数在等值情况下返回 false。

**eg 1. 静态数组排序**

```C++
// arrays - 数组名（首地址），size - 数组大小
sort(arrays, arrays + size);                 // 默认升序排序
sort(arrays, arrays + size, less<int>());    // 升序排序
sort(arrays, arrays + size, greater<int>()); // 降序排序
```

**eg 2. 动态数组排序**

```C++
sort(v.begin(), v.end());                 // 默认升序排序
sort(v.begin(), v.end(), less<int>());    // 升序排序
sort(v.begin(), v.end(), greater<int>()); // 降序排序
```

## 列表

### 声明及初始化

使用 `list (STL list)` 时，需引入头文件 `<list>`。

```C++
#include <list>

/* eg 1 initialize */

// 初始化一个 int 类型的空列表 nums
list<int> nums;

// 初始化一个大小为 n 的列表 nums，列表中的值默认都为 0
list<int> nums(10);

// 初始化一个大小为 n 的列表 nums，其值全都为 2
list<int> nums(n, 2);

/* eg 2 make a copy */
list<int> list_1(list_2.begin(), list_2.end());
list<int> list_1(list_2);

/* eg 3 cast an array to a list */
int a[5] = {0, 1, 2, 3, 4};
list<int> nums(a, *(&a + 1));
```

### 遍历列表

**eg 1. 基于范围的 for 循环遍历**

```C++
for (int &val : my_list) {
    cout << val << endl;
}
```

**eg 2. 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto i = my_list.begin()
for (list<int>::iterator i = my_list.begin(); i != my_list.end(); ++i) {
    cout << *i << endl;
}
```

> **注意：** `list` 容器未提供下标操作符 `[]` 和 `at()` 成员函数，也没有提供 `data()` 成员函数，故不支持随机访问，不能通过下标循环遍历。

### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type size();

// 添加元素
void push_back(const value_type& val);
void push_front(const value_type& val);
void emplace_back(Args&&... args);
void emplace_front(Args&&... args);

// 返回元素引用
reference back();
reference front();

// 删除元素
void pop_back();
void pop_front();

// 将列表 size 设置为 0，但 capacity 不变（注意：不能用于清空列表）
void clear();

// 插入单个元素
iterator insert(const_iterator position, const value_type& val);

// 填充 n 个元素
iterator insert(const_iterator position, size_type n, const value_type& val);

// 插入 [first,last) 范围内的所有元素
iterator insert(const_iterator position, InputIterator first, InputIterator last);
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

**eg 1. 下标 for 循环遍历**

```C++
for (int i = 0; i < a.size(); ++i) {
    cout << a[i] << " ";
}
```

**eg 2. 基于范围的 for 循环遍历**

```C++
for (char c : a) {
    cout << c << "  ";
}
```

**eg 3. 基于迭代器遍历**

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

/* insert */

// 在迭代器 p 指向的元素之前创建一个值为 t 的元素，返回指向新添加的元素的迭代器
str.insert(p, t);

// 在迭代器 p 指向的元素之前插入 n 个值为 t 的元素，返回指向新添加的第一个元素的迭代器
str.insert(p, n, t);

// 将迭代器 b 和 e 指定范围内的元素插入到迭代器 p 指向的元素之前，返回指向新添加的第一个元素的迭代器
str.insert(p, b, e);
```

### 扩展操作

#### 字符串逆转

使用 `reverse()` 函数时，需引入头文件 `<algorithm>`。

```C++
#include <algorithm>
void reverse<_BidIt>(const _BidIt _First, const _BidIt _Last);

string str = "hello world";

// 逆转整个字符串
reverse(str.begin(), str.end()); // dlrow olleh

// 逆转部分字符串
reverse(str.begin(), str.begin() + 5); // olleh world
```

#### 处理 string 对象中的字符

* `isalpha(c)` 当 c 是字母时为真
* `isdigit(c)` 当 c 是数字时为真
* `islower(c)` 当 c 是小写字母时为真
* `isupper(c)` 当 c 是大写字母时为真
* `tolower(c)` 如果 c 是大写字母，返回对应的小写字母；否则原样返回 c
* `toupper(c)` 如果 c 是小写字母，返回对应的大写字母；否则原样返回 c

#### 字符串分割

C++ 中没有直接的 `split` 函数，字符串分割可借助以下方法实现：

**eg 1. 利用 C `strtok` 函数**

利用 `strtok` 函数实现字符串分割时，需引入 C 头文件 `<string.h>`。该函数原型如下：

```C
char* strtok(char *str, const char *delim);
```

返回值：从 `str` 开头开始的一个个被分割的串。当 `str` 中的字符查找到末尾时，返回 `NULL`，如果查找不到 `delim` 中的字符时，则返回当前 `strtok` 中的字符串的指针。

在首次调用 `strtok` 时，需传递字符串参数 `str`，往后的调用则将参数 `str` 设置成 `NULL`。注意，由于当 `strtok` 发现对应分隔符时，会将该字符改为 `\0` 字符，即传入 `str` 参数会被修改。故在调用该方法前，需要将字符串复制一份再传入。

```C++
vector<string> split(const string &str, const char *delim) {
    vector<string> res;
    char *buffer = new char[str.size() + 1]; // 定义 C 风格字符串，用于修改
    strcpy(buffer, str.c_str());             // 复制字符串
    char *p = strtok(buffer, delim);         // 首次调用，传入复制字符串

    while (p) {
        res.emplace_back(p);
        p = strtok(NULL, delim);             // 再次调用，修改 str 参数为 NULL
    }

    delete buffer;
    return res;
}

int main() {
    string test = "a,b,c,d,e";
    vector<string> res = split(test,  ',');
    for (auto &val : res) {
        cout << val << " "; // 输出：a b c d e
    }
    return 0;
}
```

**eg 2. 利用 C++ 流**

利用 `stringstream` + `getline` 方法实现字符串分割时，需引入头文件 `<sstream>`。

```C++
vector<string> split(const string &str, const char delim) {
    vector<string> res;
    stringstream ss(str); // 读取 str 到字符串流中
    string tmp;

    /* 使用 getline 方法从字符串流中读取，当读取到分隔符时停止
    注意：getline 默认可以读取空格 */
    while (getline(ss, tmp, delim)) {
        res.emplace_back(tmp);
    }

    return res;
}

int main() {
    string test = "a,b,c,d,e";
    vector<string> res = split(test,  ',');
    for (auto &val : res) {
        cout << val << " "; // 输出：a b c d e
    }
    return 0;
}
```

### 类型转换

使用以下转化方法时，需引入头文件 `<string>`，并使用 `std` 命名空间。

#### 数值 转换为 string

* `to_string`  : numerical value to string

```C++
string to_string (numeric_type val);
```

#### string 转换为 数值

* `stoi`   : string to integer
* `stol`   : string to long int
* `stoul`  : string to unsigned integer
* `stoll`  : string to long long
* `stoull` : string to unsigned long long
* `stof`   : string to float
* `stod`   : string to double
* `stold`  : string to long double

```C++
numeric_type sto* (const string& str, [...]);
```

## 队列

使用队列 `queue` 或 优先队列 `priority_queue` 时，需引入头文件 `<queue>`。

### 普通队列

#### 声明及初始化

```C++
#include <queue>

queue<int> q;
```

#### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type size();

// 将元素加入队尾
void push(const value_type& val);

// 返回队头元素的引用
value_type& front();

// 删除队头元素
void pop();
```

> **注意：** `pop()` 方法为 `void` 类型，不会返回被删除的元素，所以元素出队的一般做法为：
> `int val = q.front(); q.pop();`

### 双端队列

#### 声明及初始化

```C++
#include <deque>

deque<int> dq;
```

#### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type size();

// 添加元素
void push_back(const value_type& val);
void push_front(const value_type& val);
void emplace_back(Args&&... args);
void emplace_front(Args&&... args);

// 返回元素引用
reference back();
reference front();

// 删除元素
void pop_back();
void pop_front();

// 将队列 size 设置为 0，但 capacity 不变（注意：不能用于清空队列）
void clear();
```

`deque` 拥有 `begin()`、`end()` 方法，可以通过迭代器进行遍历。

### 优先队列

#### 声明及初始化

定义：`priority_queue<Type, Container, Functional>`

* `Type`：数据类型
* `Container`：容器类型（默认 vector）
* `Functional`：比较方法

```C++
#include <queue>

// 小顶堆 - 升序队列
priority_queue<int, vector<int>, greater<int>> q;

// 大顶堆 - 降序队列
priority_queue<int, vector<int>, less<int>> q;
```

> **注意：**
> **①** 一般只有在使用自定义的数据类型时，才需要传入这三个参数，使用基本数据类型时，只需传入数据类型，**默认为大顶堆 - 降序队列。**
>
> **②** 自定义排序的比较与 sort 方法相反。

```C++
/* 对于基础类型，默认为大顶堆 - 降序队列
等价于 priority_queue<int, vector<int>, less<int>> q; */
priority_queue<int> q; 
```

#### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type size();

// 将元素加入队列，并排序
void push(const value_type& val);

// 返回队头元素的引用
value_type& top();

// 删除队头元素
void pop();
```

> **注意：** 队列没有 vector 的 begin() 方法，故不能使用 基于范围的 for 循环遍历 和 基于迭代器的遍历。只能通过 while 判断 empty() 方法 或 根据 size() 大小进行 for 循环 来进行遍历。

#### 自定义排序

**eg 1. 重载仿函数 ()**

```C++
struct cmp {
    bool operator () (ListNode* a, ListNode* b) {
        return a->val > b->val; // 升序，小顶堆
    }
};

priority_queue<ListNode*, vector<ListNode*>, cmp> pq; 
```

**eg 2. 重载 operator <**

```C++
bool operator < (ListNode a, ListNode b){
    return a.val < b.val; // 降序，大顶堆
}

priority_queue<ListNode> pq; 
```

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

TODO set multiset

### 声明及初始化

使用集合 `unordered_set` 时，需引入头文件 `<unordered_set>`。

```C++
#include <unordered_set>

// 初始化一个空的 set
unordered_set<int> set;

// 通过初始化列表中的元素，对其进行初始化
unordered_set<char> s{'a', 'b', 'c', 'd', 'e', 'f', 'g'};
```

### 遍历集合

**eg 1. 基于范围的 for 循环遍历**

```C++
// type val
for (auto val : set) {
    cout << val << endl;
}
```

> **注意：** 不能通过基于范围的 for 循环遍历修改集合元素，即使采用 for (auto &val : set) 形式，set 返回的将是 const type &val，即指向常量的引用，同样不能修改。

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

// 将集合 size 设置为 0，但 capacity 不变（注意：不能用于清空集合）
void clear();
```

## 哈希表

TODO map multimap

### 声明及初始化

使用哈希表 `unordered_map` 时，需引入头文件 `<unordered_map>`。

```C++
#include <unordered_map>

// 初始化一个空的 map
unordered_map<int, int> mapping;

// 通过初始化列表中的键值对，对其进行初始化
unordered_map<char, char> pairs = {
    {')', '('},
    {']', '['},
    {'}', '{'}
};
```

### 遍历哈希表

**eg 1. 基于范围的 for 循环遍历**

```C++
// pair<const type, type> item
for (auto item : mapping) {
    cout << item.first << " " << item.second << endl;
}
```

**eg 2. 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto iter = mapping.begin()
for (unordered_map<int, string>::iterator iter = mapping.begin(); iter != mapping.end(); ++iter) {
    cout << iter->first << " " << iter->second << endl;
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

// 将哈希表 size 设置为 0，但 capacity 不变（注意：不能用于清空哈希表）
void clear();
```

> **注意：** 用 `[]` 访问其中的键 `key` 时，如果 `key` 不存在，则会自动创建 `key`，对应的值为该值类型的默认值。

## pair

### 声明及初始化

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

### 基本操作

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

## 其他操作

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
max(a, b, comp) // b: -2

bool comp(const int a, const int b) {
    return a < b * b;
}

// initializer list (3) 
max({a, b, c}); // c: 3
```

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

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [C++ 数组](https://www.runoob.com/cplusplus/cpp-arrays.html)
* [C++ 数组和vector的基本操作](https://www.cnblogs.com/HL-space/p/10546585.html)
* [C++ 中使用std::sort自定义排序规则时要注意的崩溃问题](https://blog.csdn.net/albertsh/article/details/119523587)
* [C++ 多维数组的遍历以及初始化](https://blog.csdn.net/anlian523/article/details/90549379)
* [C++ 迭代器（STL迭代器）iterator详解](http://c.biancheng.net/view/338.html)
* [C++ string类（C++字符串）完全攻略](http://c.biancheng.net/view/400.html)
* [C++ 优先队列(priority_queue)用法详解](https://blog.csdn.net/weixin_36888577/article/details/79937886)
* [C++ STL pair用法详解](http://c.biancheng.net/view/7169.html)
* [C++ 中实现类似split()的字符串分割函数](https://guopengzhen.com/%E7%A8%8B%E5%BA%8F%E7%8C%BF%E7%9A%84%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/8319/)
