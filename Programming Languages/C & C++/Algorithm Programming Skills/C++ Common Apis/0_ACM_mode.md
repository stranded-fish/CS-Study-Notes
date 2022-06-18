# C++ ACM 编程模式总结

本文主要总结了 C++ ACM 编程模式的代码模板与常见的输入/输出处理。

目录：

- [C++ ACM 编程模式总结](#c-acm-编程模式总结)
  - [代码模板](#代码模板)
  - [输入/输出处理](#输入输出处理)
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

## 输入/输出处理

## 参考链接

* [SamZhangQingChuan/Templates-for-Competitive-Programming](https://github.com/SamZhangQingChuan/Templates-for-Competitive-Programming/blob/master/%E5%A4%B4%E6%96%87%E4%BB%B6%E5%8F%8A%E6%B5%8B%E8%AF%95/Template.cpp)
