# C++ ACM 编程模式总结

本文主要总结了 C++ ACM 编程模式的代码模板与常见的输入/输出处理。

目录：

- [C++ ACM 编程模式总结](#c-acm-编程模式总结)
  - [代码模板](#代码模板)
  - [基本输入](#基本输入)
    - [单组输入](#单组输入)
    - [多组输入 - 固定数量](#多组输入---固定数量)
    - [多组输入 - 直到读至输入文件末尾](#多组输入---直到读至输入文件末尾)
    - [多组输入 - 直到读至特殊输入](#多组输入---直到读至特殊输入)
  - [字符串输入](#字符串输入)
    - [单个字符串不含默认分隔符（空格、制表符、回车换行）](#单个字符串不含默认分隔符空格制表符回车换行)
    - [单个字符串含有空格、制表符，但不含回车换行](#单个字符串含有空格制表符但不含回车换行)
    - [逐个字符输入](#逐个字符输入)
    - [单行输入 - 固定单行字符串数量](#单行输入---固定单行字符串数量)
    - [多行输入 - 不定单行字符串数量](#多行输入---不定单行字符串数量)
    - [多行输入 - 单行含分隔符](#多行输入---单行含分隔符)
  - [格式化输出](#格式化输出)
    - [不同进制输出](#不同进制输出)
    - [保留指定的有效位数与小数](#保留指定的有效位数与小数)
    - [格式化输出宽度](#格式化输出宽度)
  - [参考链接](#参考链接)

## 代码模板

```C++
/* ----------------- 头文件 ----------------- */

// C
#include <cstdio>
#include <cstdlib>
#include <cassert>
#include <cstring>
#include <cctype>
#include <ctime>
#include <cmath>

// C++

// IO
#include <iostream>
#include <iomanip>
#include <sstream>

// 算法
#include <algorithm>
#include <numeric>

// 容器
#include <vector>
#include <list>
#include <string>
#include <queue>
#include <deque>
#include <stack>
#include <set>
#include <unordered_set>
#include <map>
#include <unordered_map>
#include <utility>
#include <bitset>

// 其他
#include <functional>
#include <random>

using namespace std;

/* ----------------- IO 加速 ----------------- */
#define FastIO ios::sync_with_stdio(false); std::cin.tie(nullptr); std::cout.tie(nullptr);


/* ----------------- 定义别名 ----------------- */
using ll = long long;
using ull = unsigned long long;
using db = double;
using ld = long double;
using pii = pair<int, int>;
using pll = pair<ll, ll>;
using pull = pair<ull, ull>;

/* ----------------- 常量定义 ----------------- */
const double EPS = 1e-6;
const int INF = 1 << 29;
const int MOD = 1e9 + 7;
const int MAX_N = 100;

/* ----------------- debug 输出 ----------------- */
string to_string(const string &s) { return '"' + s + '"'; }

string to_string(const char *s) { return to_string((string) s); }

string to_string(char c) { return '\'' + string(1, c) + '\''; }

string to_string(bool b) { return (b ? "true" : "false"); }

template<typename A, typename B>
string to_string(pair<A, B> p) { return "(" + to_string(p.first) + ", " + to_string(p.second) + ")"; }

template<typename T>
string to_string(T t) {
    bool first = true;
    string res = "{";
    for (const auto &x: t) {
        if (!first) res += ", ";
        first = false;
        res += to_string(x);
    }
    res += "}";
    return res;
}

void debug() { cerr << endl; }

template<typename Head, typename... Tail>
void debug(Head H, Tail... T) {
    cerr << to_string(H) << " ";
    debug(T...);
}

/* ----------------- 字符串分割 ----------------- */
vector<string> split(const string &str, const char &delim) {
    vector<string> res;
    stringstream ss(str);
    string tmp;
    while (getline(ss, tmp, delim)) { if (!tmp.empty()) res.push_back(std::move(tmp)); }
    return res;
}

// 示例
int main() {
    // 启用 IO 加速
    // 注意: 启用后不能混用 cin/cout 与 scanf/printf
    FastIO

    int a = 3;
    db b = 3.45;
    ld c = 123.456;
    char d = 'x';
    string e = "123";

    // 输出: 3 3.450000 123.456000 'x' "123"
    debug(a, b, c, d, e);

    pii p1 = make_pair(1, 2);
    pll p2 = make_pair(LONG_LONG_MIN, LONG_LONG_MAX);
    pull p3 = make_pair(ULONG_LONG_MAX, ULONG_LONG_MAX);

    // 输出: (1, 2) (-9223372036854775808, 9223372036854775807) (18446744073709551615, 18446744073709551615)
    debug(p1, p2, p3);

    vector<string> arr;
    string str = "123,456,789";

    // 字符串分割
    arr = split(str, ',');

    unordered_set<int> s;
    s.insert(1);
    s.insert(2);
    unordered_map<int, string> mappings;
    mappings[1] = "hello";
    mappings[2] = "world";

    // 输出: {"123", "456", "789"} {2, 1} {(2, "world"), (1, "hello")}
    debug(arr, s, mappings);

    return 0;
}
```

## 基本输入

### 单组输入

```C++
int main() {
    int a, b;
    cin >> a >> b;
    cout << a + b << '\n';
    return 0;
}
```

### 多组输入 - 固定数量

示例：[A+B(2)](https://ac.nowcoder.com/acm/contest/5657/B)

```C++
int main() {
    int t, a, b;
    cin >> t;
    while (t--) {
        cin >> a >> b;
        cout << a + b << '\n';
    }
}
```

### 多组输入 - 直到读至输入文件末尾

示例：[A+B(1)](https://ac.nowcoder.com/acm/contest/5657/A)

```C++
int main() {
    int a, b;
    while (cin >> a >> b) {
        cout << a + b << '\n';
    }
    return 0;
}
```

### 多组输入 - 直到读至特殊输入

示例：[A+B(3)](https://ac.nowcoder.com/acm/contest/5657/C)

```C++
int main() {
    int a, b;
    while (cin >> a >> b && (a != 0 || b != 0)) {
        cout << a + b << '\n';
    }
    return 0;
}
```

## 字符串输入

### 单个字符串不含默认分隔符（空格、制表符、回车换行）

```C++
int main() {
    string a, b;
    cin >> a >> b;         // 输入: 123 456
    cout << a << " " << b; // 输出: 123 456
}
```

### 单个字符串含有空格、制表符，但不含回车换行

```C++
string a;
cin >> a;        // 输入: how are you!
cout << a;       // 输出: how

string b;
getline(cin, b); // 输入: how are you!
cout << b;       // 输出: how are you!
```

### 逐个字符输入

`std::istream::get`：从流中提取字符，作为未格式化的输入。

函数签名：

```C++
// single character (1)
// 从流中提取单个字符
int get();
istream& get (char& c);

// c-string (2)
// 从流中提取字符并将它们作为 c 字符串存储在 s 中，
// 直到提取 (n-1) 个字符或遇到分隔符：分隔符是换行符 ('\n') 或 delim（如果指定了此参数）
// 如果找到分隔符，则不会从输入序列中提取，而是保留在那里作为要从流中提取的下一个字符
istream& get (char* s, streamsize n);
istream& get (char* s, streamsize n, char delim);

// stream buffer (3)
// 从流中提取字符并将它们插入到由流缓冲区对象 sb 控制的输出序列中，
// 一旦这样的插入失败或在输入序列中遇到定界字符（定界字符是换行符）就停止，
// 如果指定了 delim 参数，则为字符 delim
istream& get (streambuf& sb);
istream& get (streambuf& sb, char delim);
```

示例：

```C++
// single character (1)
char c;
while (c = cin.get()) {...}
while (cin.get(c)) {...}

// c-string (2)
char ch[20];
cin.get(ch, 10, '.'); // 读取前 9 个字符或直到分隔符 '.'
```

### 单行输入 - 固定单行字符串数量

示例：[字符串排序(1)](https://ac.nowcoder.com/acm/contest/5657/H)

```C++
int main() {
    int t;
    cin >> t;
    vector<string> arr;
    string tmp;
    while (t--) {
        cin >> tmp;
        arr.push_back(std::move(tmp));
    }
    ...
    return 0;
}
```

### 多行输入 - 不定单行字符串数量

示例：[字符串排序(2)](https://ac.nowcoder.com/acm/contest/5657/I)

```C++
int main() {
    string str;
    vector<string> arr;
    while (cin >> str) {
        arr.push_back(std::move(str));
        if (cin.get() == '\n') {
            sort(arr.begin(), arr.end());
            for (const auto &s : arr) cout << s << ' ';
            cout << '\n';
            arr.clear();
        }
    }
    return 0;
}
```

### 多行输入 - 单行含分隔符

示例：[字符串排序(3)](https://ac.nowcoder.com/acm/contest/5657/J)

```C++
vector<string> split(const string &str, const char &delim) {
    vector<string> res;
    stringstream ss(str);
    string tmp;
    while (getline(ss, tmp, delim)) { if (!tmp.empty()) res.push_back(std::move(tmp)); }
    return res;
}

int main() {
    string str;
    while (getline(cin, str)) {
        vector<string> arr = split(str, ',');
        sort(arr.begin(), arr.end());
        for (int i = 0; i < arr.size(); ++i) {
            if (i != arr.size() - 1) {
                cout << arr[i] << ',';
            } else cout << arr[i] << '\n';
        }
    }
    return 0;
}
```

## 格式化输出

ACM 编程模式的输入和输出是相互独立的，故每当处理完一组测试数据，就应当按照题目要求进行相应的输出操作，而不必将所有输出结果保存起来一起输出。

### 不同进制输出

**eg 1. C++ cout**

```C++
int val = 15;

// 默认算子 *dec - 以 十进制 形式输出整数
cout << val << '\n';

// 流操作算子 oct - 以 八进制 形式输出整数
cout << oct << val << '\n'; // 17
    
// 流操作算子 hex - 以 十六进制 形式输出整数
cout << hex << val << '\n'; // f
    
// 利用 bitset 将其转换为 二进制 形式输出整数
cout << bitset<sizeof(int) * 8>(val) << '\n'; // 32 bit: 00000000000000000000000000001111
cout << bitset<8>(val) << '\n';               // 8 bit : 00001111
```

**eg 2. C printf**

```C
int val = 15;

printf("%d\n", val); // 十进制   15
printf("%o\n", val); // 八进制   17
printf("%x\n", val); // 十六进制 f
```

### 保留指定的有效位数与小数

**eg 1. C++ cout**

```C++
// 注意：以下保留操作，末位均会四舍五入

double val = 1234.456789;

// 默认保留 6 位有效数字
cout << val << '\n'; // 1234.46

// setprecision(n) 设置保留 n 位有效数字
cout << setprecision(8) << val << '\n'; // 1234.4568

// a. fixed + setprecision(n) 设置保留小数点后 n 位，如果 n ＞ 小数点位数，则添 0 补齐
cout << fixed << setprecision(2) << val << '\n';  // 1234.46
cout << fixed << setprecision(10) << val << '\n'; // 1234.4567890000

// b. 预先设置
cout.setf(ios::fixed);
cout << setprecision(2);
cout << val << '\n'; // 1234.46
```

**注意：** 使用流算子之后，会对后面所有操作生效。

**eg 2. C printf**

```C
// 注意：以下保留操作，末位均会四舍五入

double val = 1234.123456789;

// 默认保留小数点后 6 位
printf("%f\n", val); // 1234.123457

// 通过 %.nf 格式化控制符，设置保留小数点后 n 位，如果 n ＞ 小数点位数，则添 0 补齐
printf("%.2f\n", val);  // 1234.12
printf("%.10f\n", val); // 1234.1234567890
```

### 格式化输出宽度

**eg 1. C++ cout**

```C++
int a = 1, b = 12;

// 设置输出宽度 5，默认右对齐
cout << setw(5) << a << endl;
cout << setw(5) << b << endl;

// 输出
//    1
//   12

// 设置输出宽度 5，不足位数用 * 补齐
cout << setfill('*') << setw(5) << a << endl;
cout << setfill('*') << setw(5) << b << endl;

// 输出
// ****1
// ***12
```

**eg 2. C printf**

```C
int a = 1, b = 12;

// 设置输出宽度 5，右对齐
printf("%5d\n", a);
printf("%5d\n", b);
    
// 输出
//     1
//    12

// 设置输出宽度 5，不足位数用 0 补齐
printf("%05d\n", a);
printf("%05d\n", b);
    
// 输出
// 00001
// 00012
```

## 参考链接

* [SamZhangQingChuan/Templates-for-Competitive-Programming](https://github.com/SamZhangQingChuan/Templates-for-Competitive-Programming/blob/master/%E5%A4%B4%E6%96%87%E4%BB%B6%E5%8F%8A%E6%B5%8B%E8%AF%95/Template.cpp)
* [OJ在线编程常见输入输出练习场](https://ac.nowcoder.com/acm/contest/5657#question)
* [ACM竞赛之输入输出（以C与C++为例）](https://cloud.tencent.com/developer/article/1077645)
* [C++ get()函数读入一个字符](http://c.biancheng.net/cpp/biancheng/view/2231.html)
* [std::istream::get](https://m.cplusplus.com/reference/istream/istream/get/)
* [C++ cout格式化输出（输出格式）完全攻略](http://c.biancheng.net/view/275.html)
