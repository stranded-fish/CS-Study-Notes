# More Effective C++ 3. 异常

本文主要为 More Effective C++ 条款 9 - 15 的学习与总结。

目录：

- [More Effective C++ 3. 异常](#more-effective-c-3-异常)
  - [条款 9：利用 destructors 避免泄漏资源](#条款-9利用-destructors-避免泄漏资源)
  - [条款 10：在 constructors 内阻止资源泄漏（resource leak）](#条款-10在-constructors-内阻止资源泄漏resource-leak)
  - [条款 11：禁止异常（exceptions）流出 destructors 之外](#条款-11禁止异常exceptions流出-destructors-之外)
  - [条款 12：了解 “抛出一个 exception” 与 “传递一个参数” 或 “调用一个虚函数” 之间的差异](#条款-12了解-抛出一个-exception-与-传递一个参数-或-调用一个虚函数-之间的差异)
  - [条款 13：以 by reference 方式捕捉 exceptions](#条款-13以-by-reference-方式捕捉-exceptions)
  - [条款 14：明智运用 exception specifications](#条款-14明智运用-exception-specifications)
  - [条款 15：了解异常处理（exception handling）的成本](#条款-15了解异常处理exception-handling的成本)
  - [参考资料](#参考资料)

## 条款 9：利用 destructors 避免泄漏资源

> **问题：** 当执行某个函数时，该函数会创建一个新的 heap object，并在函数结束前调用 delete 进行销毁，然而在执行 delete 之前该函数可能会抛出异常。
> **后果：** 该函数内位于抛出异常之后的语句都会被跳过，不再执行，最终造成资源泄漏。

**解决方法：** 为防止资源泄漏，可将资源封装在 RAII 对象中，以对象管理资源，使其在构造函数中获得资源并在析构函数中释放资源。

> 更多内容可参见《Effective C++》3. 资源管理 - 条款 13：以对象管理资源。

## 条款 10：在 constructors 内阻止资源泄漏（resource leak）

> **问题：** 在 constructors 中创建新的 heap object，并计划在 destructor 中释放资源，但 constructors 内可能发生异常。
> **后果：** 由于 C++ 只会析构已构造完成的对象，对象只有在其 constructors 执行完毕后才算是完全构造成功，所以，在对象构造过程中发生异常，将不会调用其 destructor，最终造成资源泄漏。

**解决方法：**

* 利用 try catch 在 constructors 中捕捉异常并处理。
* 将待创建的 heap object 交由局部 RAII 对象来管理（推荐）。

## 条款 11：禁止异常（exceptions）流出 destructors 之外

**优点：**

* 可以避免 terminate 函数在 exception 传播过程的栈展开（stack-unwinding）机制中被调用；
  * terminate 函数又会调用 abort 函数，abort 函数会导致程序直接中止。
* 可以协助确保 destructors 完成其应该完成的所有事情。
  * 一旦异常在 destructors 中被抛出，该异常抛出之后的语句将无法被执行。

## 条款 12：了解 “抛出一个 exception” 与 “传递一个参数” 或 “调用一个虚函数” 之间的差异

**相同点：**

* 函数参数传递和 exceptions 的传递方式均有三种：by value、by reference、by pointer。

**不同点：**

* 调用函数后，控制权最终会回到调用端（除非函数失败以至无法返回）。但抛出 exception 后，控制权不会再回到抛出端。
* exception objects 总是会被复制。但函数参数传递则不一定。
  * 任何 exception 都会产生一个临时对象，即使采用 by reference 方式也需要付出该临时对象的构造成本，而如果是 by value 则需要付出两次构造成本。推荐选择 by reference（参见条款 13）。
  * 如果 catch 语句中还有可能抛出异常，则应该使用 `throw;` 而不是 `throw w;` 以避免产生新的 exception objects。
* 被抛出成为 exception 的对象，其被允许的类型转换动作，比函数传参更少。
  * exception 与 catch 子句相匹配的过程中，仅有两种转换可以发生：
    * 继承架构中的类转换，即一个针对 base class 编写的 catch 子句，可以处理类型为 derived class 的 exception。
    * 有型指针 到 无型指针 的转换，即针对 `void*` 指针编写的 catch 子句，可以捕捉任何指针类型的 exception。
* catch 子句以其出现在源代码中的顺序被编译器检验比对，其中第一个匹配成功者便被执行。然而对象调用虚函数，则是选择与对象类型最佳匹配的函数，不论其源码顺序。
  * 故捕捉范围较小的 catch 应该放在范围较广的 catch 子句之前，例如：处理 derived class 的 catch 子句应该放在处理 base class 子句之前。

## 条款 13：以 by reference 方式捕捉 exceptions

**优点：**

* 相比 by pointer：
  * 避免指向已经被销毁的局部对象。
  * 如果是 heap object 将不用考虑资源释放问题。
* 相比 by value：
  * 避免复制两次（参见条款 12）。
  * 避免引起切割问题，即 derived class exception object 被捕捉并被视为 base object，失去了其派生部分。

## 条款 14：明智运用 exception specifications

> **问题：** 函数抛出了一个并未列于其 exception specifications 的 exception。
> **后果：** 该错误会在运行时被检验出来，导致特殊函数 unexcepted 被自动调用，unexcepted 函数默认行为是调用 terminate，最终导致程序直接中止。

**预防方法：**

* 避免将 exception specifications 放在 “需要类型自变量” 的 templates 身上。
* 如果 A 函数内调用了 B 函数，而 B 函数无 exception specifications，那么 A 函数本身也不要设定 exception specifications。
* 处理系统可能抛出的 exceptions。

**处理方法：**

* 以特定类型的 exceptions 取代所有非预期的 exceptions。

## 条款 15：了解异常处理（exception handling）的成本

为了能够在运行期处理 exceptions，程序必须做大量记录工作，这些记录工作会付出相应的成本，即使未使用关键词 try、throw 或 catch，也会付出一些成本。

为了让 exception 的相关成本最小化，只要不是必须用 exception 的地方，就应该明确告知编译器（C++ 11 可以使用 noexcept 指定），以便编译器进行相关优化（但如果在运行时，noexecpt 函数向外抛出了异常，程序仍会调用 terminate 中止程序）。

**noexcept 使用建议：**

* 析构函数。因为析构函数不允许抛出异常（参见条款 11）。
* 构造函数（普通、拷贝、移动），赋值运算符。
* 确保一定不会抛出异常的函数。
  * 如：内置类型 int、double 这类的 setter 和 getter。

## 参考资料

* 《More Effective C++：35个改善编程与设计的有效方法》（中文版）
