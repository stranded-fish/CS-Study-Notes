# C++ 引用与指针的区别

## 从高级语言层面上

引用是变量的别名，与一个变量相绑定，避免了非空判断

## 从底层原理来讲

引用从使用者角度来说，可以视为 常量指针（指针本身是个常量），实际上大多数编译器也确实通过指针来实现引用，然后在其基础上进行了封装，本质是语法糖，避免了手动 &取地址 和 *解引用

常量：
    指针自身是常量：`constant pointer    -  int *const p;`  (顶层 const，表示指针本身是个常量)
    被指物是常量：`pointer to constant -  const int *p;` （底层 const，表示指针所指的对象是个常量，但是实际上没有规定其所指对象必须是个常量，只是要求不能通过这个指针进行修改罢了）

```C++
int a = 5;
int &ref = a;
cout << ref << endl;

// 等价效果
int *const p = &a;  // 取地址
cout << *p << endl; // 解引用
```
