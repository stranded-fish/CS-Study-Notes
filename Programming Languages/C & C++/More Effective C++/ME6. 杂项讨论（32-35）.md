# More Effective C++ 6. 杂项讨论

本文主要为 More Effective C++ 条款 32 - 35 的学习与总结。

目录：

- [More Effective C++ 6. 杂项讨论](#more-effective-c-6-杂项讨论)
  - [条款 32 在未来时态下发展程序](#条款-32-在未来时态下发展程序)
  - [条款 33 将非尾端类（non-leaf classes）设计为抽象类（abstract classes）](#条款-33-将非尾端类non-leaf-classes设计为抽象类abstract-classes)
  - [条款 34 如何在同一个程序中结合 C++ 和 C](#条款-34-如何在同一个程序中结合-c-和-c)
  - [条款 35 让自己习惯于标准 C++ 语言](#条款-35-让自己习惯于标准-c-语言)
  - [参考资料](#参考资料)

## 条款 32 在未来时态下发展程序

在未来时态下发展程序，就是接受 “事情总会改变” 的事实，并准备应对之道：

* 提供完整的 classes，即使某些部分目前用不到。这样可以使得当新的需求到来，不太需要回头去修改那些 classes。
  * 为每一个 class 处理 assignment 和 copy construction 动作，即使当前没人使用，不代表以后没人用。
  * 如果一些函数不易完成，请将它们声明为 private（或明确拒绝 `=delete`），以避免任何人不经意调用 “编译器自动生成，但行为却错误” 的版本（通常发生于 default assignment operators 和 copy construct 上 —— 参见条款 E6）。
* 设计接口，使有利于共同的操作行为，阻止共同的错误。
  * 参见条款 E18。
* 尽量使代码一般化（泛化），除非有不良的巨大后果。

“未来式思维” 可增加代码的重用性、加强其可维护性、使其更健壮，并促使在一个 “改变实乃必然” 的环境中有着优雅的改变。

## 条款 33 将非尾端类（non-leaf classes）设计为抽象类（abstract classes）

一般性法则：继承体系中的 non-leaf（非尾端，即被其他类所继承）类应该是抽象类。

## 条款 34 如何在同一个程序中结合 C++ 和 C

如果打算在同一个程序中混用 C++ 和 C，那么需要记住以下守则：

* 确定你的 C++ 和 C 编译器产生兼容的目标文件（object files）。
* 将双方都使用的函数声明为 extern "C"。
* 如果可能，尽量在 C++ 中撰写 main。
* 总是以 `delete` 删除 `new` 返回的内存；总是以 `free` 释放 `malloc` 返回的内存。
* 将两个语言间的 “数据结构传递” 限制于 C 所能了解的形式；
  * C++ struct 内存布局规则与 C 一致，如果不涉及虚函数，同一个 struct 定义式所定义的对象，可安全地在 C++ 和 C 中传递。但如果涉及虚函数，会导致 C++ 对象采用不同的内存布局（参见条款 ME24），无法再与 C 兼容。

## 条款 35 让自己习惯于标准 C++ 语言

C++ 标准程序库特点：

* 标准程序库中的几乎每一样东西都是 template。
  * 包括 string，string 本质上是 `basic_string<char>`，由于其使用频繁，标准程序库提供了 `typydef basic_string<char> string`，string 还可由 wide chars，Unicode chars 等构成（补充：C++ 11 之后，原基于 Reference-counted（引用计数）和 COW（copy on write - 写时复制）的 string 实现已经被废弃）。
* 所有成分都位于 namespace std 内。
  * 为了在不用写出显式完全限定名的情况下使用标准程序库组件，可使用 `using namespace std;`。

STL（Standard Template Library，标准模板库）是 C++ 标准程序库中最大的组成部分。其主要包括三大模块：

* 容器（containers）：持有一系列对象。
* 迭代器（iterators）：一种类似指针的对象，使得可以像指针遍历内建数组一样，遍历 STL 容器。
* 算法（algorithms）：可作用于 STL 容器 身上的函数，以 iterators 来协助工作。

其他三大模块补充：

* 分配器（allocators）：用于 STL 容器分配内存，一般不直接调用。
* 适配器（adapters）：将一种接口适配为另一种接口。例如：queue、stack 默认为 deque 的适配器。
* 仿函数（functors）：表现为函数的重载了函数调用运算符的类对象（又称函数对象）。

## 参考资料

* 《More Effective C++：35个改善编程与设计的有效方法》（中文版）
