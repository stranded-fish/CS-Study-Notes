# More Effective C++ 2. 操作符

本文主要为 More Effective C++ 条款 5 - 8 的学习与总结。

目录：

- [More Effective C++ 2. 操作符](#more-effective-c-2-操作符)
  - [条款 5：对定制的 “类型转换函数” 保持警觉](#条款-5对定制的-类型转换函数-保持警觉)
  - [条款 6：区别 `++` / `--` 操作符的前置和后置形式](#条款-6区别------操作符的前置和后置形式)
  - [条款 7：千万不要重载 `&&` `||` 和 `,` 操作符](#条款-7千万不要重载---和--操作符)
  - [条款 8：了解各种不同意义的 new 和 delete](#条款-8了解各种不同意义的-new-和-delete)
  - [参考资料](#参考资料)

## 条款 5：对定制的 “类型转换函数” 保持警觉

隐式类型转换操作符 和 单自变量 constructors 允许编译器进行隐式类型转换。

**a. 隐式类型转换操作符：**

```C++
class Rational {
public:
  ...
  // 关键词 operator 后加上类型名称
  // 不能指定返回值类型，因为其返回值已经表现于函数名称上了
  operator double() const; // 将 Rational 转换为 double
};
```

该函数会在以下情况被自动调用：

```C++
Rational r(1, 2);
double d = 0.5 * r; // r 将被转换为 double，然后执行乘法运算
```

* **潜在问题：** 可能导致错误（非预期）的函数被调用。
* **解决方法：** 以功能对等的另一个函数取代类型转换操作符。

**b. 单自变量 constructors：**

```C++
class Name {
public:
  Name(const string &s); // 可以将 string 转换为 Name 对象
  ...
};
```

**潜在问题：** 可能导致错误（非预期）的函数被调用。

```C++
// 错误示例
template<typename T>
class Array {
public:
  Array(int size);
  ...
};
...
Array<int> a(10);
Array<int> b(20);

// 用户错误键入
if (a == b[i]) // a 原本该是 a[i]

// 编译器并不会报错，而是自动执行类型转换将 int 型 b[i] 
// 转化成大小为 b[i] 的临时数组 Array<int>(b[i])，然后与 a 进行比较。
```

**解决方法：**

* 使用 `explicit` 关键词修饰单自变量 constructors。
* 将单变量参数封装为 proxy classes，proxy classes 允许单自变量构造。

## 条款 6：区别 `++` / `--` 操作符的前置和后置形式

* `++`: increment 操作符
* `--`: decrement 操作符

**函数定义区别：**

由于重载函数是以其参数类型来区分彼此的，所以为了区分前置式和后置式，后置式入参添加了一个 int 自变量。同时由于未使用该参数，可忽略该参数名称。

```C++
class UPInt {
public:
    UPInt& operator++();         // 前置式 ++ 
    const UPInt operator++(int); // 后置式 ++

    UPInt& operator--();         // 前置式 --
    const UPInt operator--(int); // 后置式 --
    ...
};
```

* 前置式返回引用，可作为左值进行赋值。
* 后置式返回常量，只能作为右值使用。

**函数效率区别：**

```C++
UPInt &UPInt::operator++() {
    *this += 1;
    return *this;
}

const UPInt UPInt::operator++(int) {
    UPInt oldValue = *this;
    UPInt::operator++();
    return oldValue;
}
```

由上式可知后置式 `++` 代码实现中会生成一个临时对象 `oldValue`，该对象会带来额外的构造与析构成本（后置式 `--` 同理）。

同时为保证前置和后置的行为一致，后置式的实现应以其前置实现为基础。

**补充：**

编译器在某些情况下会对后置式进行优化，使最终其汇编表现与前置式相同。

## 条款 7：千万不要重载 `&&` `||` 和 `,` 操作符

不要重载 `&&` `||` 和 `,` 操作符：

* 针对 `&&` `||` 操作符的重载将使得 “函数调用语义” 取代 “骤死式（短路）语义”。
  * 任何情况下都会计算出所有表达式的值。
  * 无法控制函数的自变量的评估顺序，原本 `&&` `||` 总是从左向右评估。
* 针对 `,` 操作符同理，“函数调用语义” 将无法控制函数的自变量的评估顺序，原本 `,` 总是从左向右评估。

## 条款 8：了解各种不同意义的 new 和 delete

* new operator：创建对象于堆上。其不但分配内存，还调用 constructor。
  * `string *ps = new string("hello");`
* operator new：仅分配内存。
  * `void *rawMemory = operator new(sizeof(string));`
* placement new：在已经分配且拥有指针的内存中构造对象。

> 更多内容可参见《Effective C++》3. 资源管理（条款 16） 与 8. 定制 new 和 delete。

## 参考资料

* 《More Effective C++：35个改善编程与设计的有效方法》（中文版）
