# More Effective C++ 5. 技术

本文主要为 More Effective C++ 条款 25 - 31 的学习与总结。

目录：

- [More Effective C++ 5. 技术](#more-effective-c-5-技术)
  - [条款 25 将 constructor 和 non-member functions 虚化](#条款-25-将-constructor-和-non-member-functions-虚化)
  - [条款 26 限制某个 class 所能产生的对象数量](#条款-26-限制某个-class-所能产生的对象数量)
  - [条款 27 要求（或禁止）对象产生于 heap 之中](#条款-27-要求或禁止对象产生于-heap-之中)
  - [条款 28 Smart Pointers（智能指针）](#条款-28-smart-pointers智能指针)
  - [条款 29 Reference counting（引用计数）](#条款-29-reference-counting引用计数)
  - [条款 30 Proxy classes（替身类、代理类）](#条款-30-proxy-classes替身类代理类)
  - [条款 31 让函数根据一个以上的对象类型来决定如何虚化](#条款-31-让函数根据一个以上的对象类型来决定如何虚化)
  - [参考资料](#参考资料)

## 条款 25 将 constructor 和 non-member functions 虚化

**a. virtual constructor**

virtual constructor 可视其获得的输入，产生不同类型的对象。其常见用途之一就是从磁盘（或网络）中读取对象信息。

其中 virtual copy constructor 可视为一种特殊的 virtual constructor，其返回一个指针，指向其调用者（某对象）的一个副本。

```C++
class Base {
public:
    // virtual copy constructor 
    virtual Base *clone() const = 0;
    ...
};

class Derive_A : public Base {
public:
    virtual Derive_A *clone() const; // 重定义 clone
    ...
};

class Derive_B : public Base {
public:
    virtual Derive_B *clone() const; // 重定义 clone
    ...
};

void test(Base *ptr) {
    // 根据 ptr 动态类型返回不同对象
    Base *p = ptr->clone();
}
```

以上实现基于 derived class 重新定义其 base class 的一个虚函数时，不再需要一定得声明与原本相同的返回类型的规则。

**b. 将 non-member functions 的行为虚化**

non-member functions 无法被真正虚化，但可以让 non-member functions 的行为视其参数的动态类型而不同。

**实现方法：** 实现一个虚函数做实际工作，然后再实现一个只负责调用虚函数的非虚函数。同时为了避免函数调用成本，可将该非虚函数 inline 化。

```C++
inline ostream &operator<<(ostream &s, const Object &c) {
    return c.print(s); // 根据 c 的动态类型调用不同的 print
}
```

## 条款 26 限制某个 class 所能产生的对象数量

**a. 允许零个或一个对象**

**eg 1. getInstance 为全局函数**

```C++
class Singleton {
public:
    void doWork();
    ...
    friend Singleton &getInstance();
    Singleton& operator=(const Singleton &) = delete; // 禁止拷贝赋值运算符
private:
    Singleton() = default;
    Singleton(const Singleton &) = default;
};

Singleton &getInstance() {
    static Singleton obj; // 唯一实例对象
    return obj;
}

// 调用方式
getInstance().doWork();
```

**设计特点：**

* Singleton class 的 constructor 被声明为 private，以压制对象的诞生。
* 全局函数 getInstance 被声明为此 class 的一个 friend，致使其不受 private constructor 的约束。
* getInstance 内含一个 static Sinleton 对象，代表只会有一个 Sinleton 对象被产生出来。

**eg 2. 将 getInstance 封装在类中**

```C++
class Singleton {
public:
    void doWork();
    ...
    static Singleton &getInstance();
    Singleton& operator=(const Singleton &) = delete; // 禁止拷贝赋值运算符
private:
    Singleton() = default;
    Singleton(const Singleton &) = default;
};

Singleton &Singleton::getInstance() {
    static Singleton obj; // 唯一实例对象
    return obj;
}

// 调用方式
Singleton::getInstance().doWork();
```

**eg 3. 将 Singleton 和 getInstance 声明于全局空间中**

```C++
namespace SingletonStuff {
    class Singleton {
    public:
        void doWork();
        ...
        friend Singleton &getInstance();
        Singleton& operator=(const Singleton &) = delete; // 禁止拷贝赋值运算符
    private:
        Singleton() = default;
        Singleton(const Singleton &) = default;
    };

    Singleton &getInstance() {
        static Singleton obj; // 唯一实例对象
        return obj;
    }
}

// 调用方式 1
SingletonStuff::getInstance().doWork();

// 调用方式 2
using namespace SingletonStuff;
// 或 using SingletonStuff::getInstance;
getInstance().doWork();
```

**在函数中生成 static 对象，而非 class 拥有一个 static 对象的优点：**

* class 中拥有一个 static 对象，即使从未被用到，也会被构造和析构。而函数中拥有一个 static 对象，该对象只会在函数第一次被调用时才产生，如果该函数从未被调用那么也就不会产生。
* class static 不能明确知道其初始化时机，但函数中的 static 可以明确知道为调用时。

**b. 不同的对象构造状态**

Singleton 对象可于 3 种不同状态下生存：

* 其自身
* 派生物的 base class 成分
* 内嵌于较大的对象中

这些不同状态的呈现，将导致通过计数方式实现 “追踪目前存在的对象个数” 的意义严重混淆。

如果采用之前 **a.** 的策略，将很容易满足约束，因为 Singleton 的 constructor 是 private 的，如果没有声明任何 friend 的话，将不能被作为 base classes 或 内嵌于其他对象中。

**补充：** 以上思路同时可以用来设计：禁止任何继承，并且允许生成任意数量的对象的类。

* 禁止任何继承：将 constructor 声明为 private。
* 允许生成任意数量的对象：提供一个伪 public constructor，调用真正的 constructor。

```C++
class Object {
public:
    // 伪 constructors
    static Object *makeObject();
    static Object *makeObject(const Object &rhs);
    ...
private:
    Object() = default;
    Object(const Object &) = default;
    ...
};

Object *Object::makeObject() { 
    return new Object(); 
}

Object *Object::makeObject(const Object &rhs) {
    return new Object(rhs);
}
```

**c. 允许对象生生灭灭**

结合之前的 **对象计数** 和 **伪构造函数** 实现并未在同一时间产生一个以上 Object 对象，但可以在程序的不同地点使用不同的 Object 对象。

```C++
class Object {
public:
    class TooManyObjects : exception {}; // 定义可能抛出的异常
    ~Object() = default;
    
    // 伪 constructor
    static Object *makeObject();
    
    // 不允许复制行为
    Object(const Object &) = delete;
    Object &operator=(const Object &) = delete;
private:
    const static size_t maxObjects = 1;
    static size_t objectCnt;
    Object();
};

size_t Object::objectCnt = 0;

Object::Object() {
    // 一旦超过一个对象被申请，即抛出异常
    if (objectCnt >= maxObjects) {
        throw TooManyObjects();
    }
    ++objectCnt; // 计数 + 1
}

Object *Object::makeObject() {
    return new Object;
}

// 调用方式
Object *p = Object::makeObject(); // 间接调用 default 构造函数
Object p1 = *p;                   // 错误！copy 行为未定义
delete p;                         // 避免资源泄漏
```

**扩展：** 修改 maxObjects 值，可将该技术泛化为 “限定生成任意个数的对象”。

**d. 一个用来计算对象个数的 Base Class**

```C++
template<typename BeingCounted>
class Counted {
public:
    class TooManyObjects : exception {};           // 定义可能抛出的异常
    static int objectCount() { return objectCnt; } // 返回当前存活的对象数量

// 声明为 protected 用于继承
protected:
    Counted();
    Counted(const Counted &rhs);
    ~Counted() { --objectCnt; } // 引用计数 - 1

private:
    static int objectCnt;           // 统计当前存活的对象数量
    static const size_t maxObjects; // 最大限制数量
    
    void init();                    // 避免 constructor 重复代码
};

template<typename BeingCounted>
Counted<BeingCounted>::Counted() { init(); }

template<typename BeingCounted>
Counted<BeingCounted>::Counted(const Counted &rhs) { init(); }

template<typename BeingCounted>
void Counted<BeingCounted>::init() {
    if (objectCnt >= maxObjects) {
        throw TooManyObjects();
    }
    ++objectCnt;
}

// Object 实现文件初始化 maxObjects
const size_t Counted<Object>::maxObjects = 10;

// Object 类运用 Counted 模板
class Object : private Counted<Object> {
public:
    ...
    // 使其在 private 继承中恢复 public 访问
    using Counted<Object>::objectCount;
    using Counted<Object>::TooManyObjects;
    ...
};
```

**设计特点：**

* 采用 private 继承（参见条款 E32 E39），同时可以避免 Counted 定义 virtual 析构函数的副作用（参见条款 ME24）。
* 所有计数动作均由 `Counted<Object> constructors` 进行处理，而由于 `Counted<Object>` 是 `Object` 的一个 base classes，在 `Object constructors` 被调用之前，总是会先调用 `Counted<Object> constructors`，最终实现派生类 `Object` 可以完全无感地使用计数服务。

## 条款 27 要求（或禁止）对象产生于 heap 之中

**a. 要求对象产生于 heap 之中**

通过限制 destructor 与 constructors 的运用，便可阻止 non-heap objects 的诞生：

* 让 destructor 成为 private，再通过引入伪 destructor 来调用真正的 destructor。
* 而 constructors 仍为 public。

```C++
class Object {
public:
    // 编译器将自动生成 public default constructors
    ...
    // 伪 destructor，声明为 const 因为 const 对象也可能需要被销毁
    void destory() const { delete this; }

private:
    ~Object() = default;
};

// 调用方式
Object obj;             // 错误！企图隐式调用 private destructor
Object *p = new Object; // 正确
delete p;               // 错误！企图调用 private destructor
p->destory();           // 正确
```

* 由于 non-heap objects 会在其寿命结束时自动析构，而隐式调用析构动作现在变得不合法了，故变相禁止了创建 non-heap objects。
* 不建议选择将所有 constructors 声明为 private，因为 class 通常有多个 constructors，一旦遗忘声明某一个为 private，编译器将自动生成 public constructors。所以仅声明 destructor 为 private 较好，因为 class 仅能有一个 destructor。
* 如果需要继承：可令 destructor 为 protected，并保持其 constructors 为 public。
* 如果需要内嵌于其他对象：可改为内含一个指针，指向其 heap 对象。

**b. 禁止对象产生于 heap 之中**

heap 中的对象总是由 new 产生出来，故可以通过禁止调用 new 来实现禁止对象产生于 heap 之中：

```C++
class Object {
private:
    static void *operator new(size_t size);
    static void operator delete(void *ptr);
    ...
};

Object obj;             // 正确
Object *p = new Object; // 错误！企图调用 private new delete   
delete p;               // 错误！企图调用 private delete
```

* 如果想禁止由 Object 对象组成的数组位于 heap 内，可以将 `operator new[]` 和 `operator delete[]` 也声明为 private。
* 如果 derived class 继承 Object 且不重新声明属于自己的 `operator new/delete`，那么以上方法同时也会阻止 Object 作为 base class 成分，被 derived class 实例化到 heap 内，因为 derived class 继承的是 Object 声明的 private 版本。

## 条款 28 Smart Pointers（智能指针）

Smart Pointers（智能指针）是看起来、用起来、感觉起来都像内建指针，但提供更多机能的一种对象。

每个 Smart Pointers to T 都内含有一个 dumb pointer to T（指向该资源的内建指针），后者才是实践指针行为的真正主角。

Smart Pointers 行为：

* 构造行为：确定一个目标物（通常利用 Smart Pointers 的 constructor 自变量），然后让 Smart Pointers 内部的 dumb pointer 指向它。如果尚未决定目标物，就将内部指针设为 0，或是发生一个错误消息（抛出 exception）。
* 其他行为（复制、赋值、析构）：视该 Smart Pointers 拥有权的观念而决定。

> 更多内容可参见原文。

## 条款 29 Reference counting（引用计数）

实现 Reference counting 的一大方法是 Copy-on-Write（写时复制）：和其他对象共享一份实值，直到我们必须对自己所拥有的那一份实值进行写动作。该方式同样也是提升效率的一般化做法（缓式评估，参见条款 ME17）中的重要实现手段。

Reference counting 的实现也需要一定成本，其适用前提是：对象常常共享实值。

> 更多内容可参见原文。

## 条款 30 Proxy classes（替身类、代理类）

Proxy classes 能够帮助我们完成一些十分困难的行为，比如：

* 多维数组
* 左值/右值的区分
* 压抑隐式类型转换

> 更多内容可参见原文。

## 条款 31 让函数根据一个以上的对象类型来决定如何虚化

最一般化的方法是利用一长串的 if-else 语句来仿真虚函数，但这种方法必须知道其每一个兄弟类，并且一旦有新加入的类型，则需要修改所有涉及到的 if-else 方法，如果遗忘其中任何一个就会出错。

可以通过自行仿真虚函数表格实现。

> 更多内容可参见原文。

## 参考资料

* 《More Effective C++：35个改善编程与设计的有效方法》（中文版）
