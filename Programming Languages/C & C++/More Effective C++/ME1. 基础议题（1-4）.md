# More Effective C++ 1. 基础议题

本文主要为 More Effective C++ 条款 1 - 4 的学习与总结。

目录：

- [More Effective C++ 1. 基础议题](#more-effective-c-1-基础议题)
  - [条款 1：仔细区别 pointers 和 references](#条款-1仔细区别-pointers-和-references)
  - [条款 2：最好使用 C++ 转型操作符](#条款-2最好使用-c-转型操作符)
  - [条款 3：绝对不要以多态方式处理数组](#条款-3绝对不要以多态方式处理数组)
  - [条款 4：非必要不提供 default constructor](#条款-4非必要不提供-default-constructor)
  - [参考资料](#参考资料)

## 条款 1：仔细区别 pointers 和 references

**区别：**

* pointers 可以不指向任何对象，即，可以是 `null` 指针，但 references 必须代表一个对象。
  * 由于 references 不能为 `null`，所以在其使用前不用测试其有效性，会比 pointers 更高效。
* pointers 可以被重新赋值，但 references 只能指向它最初获得的那个对象。

**适用场景：**

* 使用 pointer：当有 “不指向任何对象” 的可能性时，或是考虑 “在不同时间指向不同对象” 的能力时。
* 使用 reference：当确定 “总是会代表某个对象”，并且 “一旦代表了该对象就不能再改变” 时。
  * 其他情况：如实现操作符 `operator[]` 时，该操作符需要返回一个能够被赋值修改的对象，此时出于可理解性，应返回 reference（假设返回 pointer，则赋值操作为：`*v[5] = 10`，会与指针数组相混淆）。

## 条款 2：最好使用 C++ 转型操作符

**C 风格 - 旧式转型缺点：**

* 缺少约束。几乎允许将任何类型转换为任何其他类型，容易出错。
* 难以辨识。旧式转型的语法结构由一对小括号加对象组成，容易与其他语法相混淆（如：函数调用）。

**推荐使用 C++ 风格 - 新式转型：**

* `static_cast<type>(expression)`：基本与 C 旧式转型拥有相同的能力与限制（除以下其他几种的能力外）。
* `const_cast<type>(expression)`：用于改变表达式中的常量性（constness）或变易性（volatileness）。
* `dynamic_cast<type>(expression)`：用于执行继承体系中 “安全向下转型或跨系转型动作”。
* `reinterpret_cast<type>(expression)`：最常用用途为转换 “函数指针” 类型。
  * 注意：该转换结果与编译器有关，不具备可移植性。

> 更多内容可参见《Effective C++》5. 实现 - 条款 27：尽量少做转型动作。

## 条款 3：绝对不要以多态方式处理数组

> **问题：** 将 derived 对象数组传递给 base 对象数组参数，并通过数组下标访问 `array[i]` 或 执行 `delete [] array` 操作。
> **后果：** 结果未定义。针对 `array[i]` 操作，编译器需要知道数组中的对象大小，然而此处会将该对象视为 base 对象，而 derived 对象通常比 base 对象大，所以基于 base 对象的指针算术表达式就会出错。

**记住：** 多态和指针算术不能混用，而数组对象几乎总是会涉及到指针的算术运算，所以数组和多态不能混用。

## 条款 4：非必要不提供 default constructor

**缺少 default constructor 的问题：**

* 生成数组：无法调用 default constructor 而报错，解决方式：
  * 使用 non-heap 数组，在定义时提供必要的自变量。
  * 使用指针数组，而非对象数组，缺点：
    * 需要记住将此数组所指向的对象删除，否则将造成内存泄漏。
    * 需要额外空间来保存指针。
  * 分配 raw memory，然后使用 placement new。
* 不适用与许多 template-base container classes。

**无意义 default constructor 的问题：**

* 增加程序复杂度。member function 需要额外判断是否为无意义变量。
* 影响程序效率。付出测试和增大可执行文件和程序库的代价。

## 参考资料

* 《More Effective C++：35个改善编程与设计的有效方法》（中文版）
