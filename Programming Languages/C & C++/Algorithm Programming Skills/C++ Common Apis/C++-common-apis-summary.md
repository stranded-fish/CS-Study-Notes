# C++ 算法练习常用 API 总结

该文总结了 C++ 刷题 **常用 API**。

总目录：

- [C++ 算法练习常用 API 总结](#c-算法练习常用-api-总结)
  - [基本数据类型](#基本数据类型)
    - [double](#double)
      - [表示格式](#表示格式)
      - [比较操作](#比较操作)
      - [输出精度](#输出精度)
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
      - [char 转化为 string](#char-转化为-string)
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
    - [set](#set)
      - [基本操作](#基本操作-7)
    - [unordered_set](#unordered_set)
      - [声明及初始化](#声明及初始化-8)
      - [遍历集合](#遍历集合)
      - [基本操作](#基本操作-8)
    - [multiset](#multiset)
  - [哈希表](#哈希表)
    - [map](#map)
      - [声明及初始化](#声明及初始化-9)
      - [遍历哈希表](#遍历哈希表)
      - [基本操作](#基本操作-9)
    - [unordered_map](#unordered_map)
      - [声明及初始化](#声明及初始化-10)
      - [遍历无序哈希表](#遍历无序哈希表)
      - [基本操作](#基本操作-10)
    - [multimap](#multimap)
  - [pair](#pair)
    - [声明及初始化](#声明及初始化-11)
    - [基本操作](#基本操作-11)
  - [算法常用 API](#其他操作)
    - [极值](#极值)
      - [max / min](#max--min)
      - [max_element / min_element](#max_element--min_element)
    - [统计](#统计)
      - [accumulate](#accumulate)
      - [count / count_if](#count--count_if)
      - [__builtin](#__builtin)
    - [搜索](#搜索)
      - [二分搜索](#二分搜索)
        - [binary_search](#binary_search)
        - [lower_bound / upper_bound](#lower_bound--upper_bound)