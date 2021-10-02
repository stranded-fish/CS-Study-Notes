# C++ 算法练习易错细节

该文总结了 C++ 刷题 **易错细节**。

目录：

- [C++ 算法练习易错细节](#c-算法练习易错细节)
  - [输入处理](#输入处理)
  - [数组操作](#数组操作)
  - [指针](#指针)
  - [溢出检查](#溢出检查)
  - [语言特性](#语言特性)
  - [参考链接](#参考链接)

## 输入处理

TODO

## 数组操作

TODO

## 指针

TODO

函数入参传递指针的时候，注意不要修改指针本身，只能修改指针所指向的内容。（因为本质上是复制传递指针的值（地址），修改该地址没有用）。

错误示范：

```C++
TreeNode*  recursion(TreeNode* node, vector<int> &nums, int start, int end) {
    node = new TreeNode(); // 此时修改 node 所指地址只能在该方法中生效（指针值传递）。
    ......
    return node;
}
```

在创建对象的时候，区分 ClassName object; 和 ClassName *object = new ClassName();
前者保存于栈空间由程序自动控制（如果是一个调用方法中，该方法返回后，则该对象内存也将被释放），后者保存于堆空间由用户控制（需手动 delete），
故如果需要在方法中持久化对象，最好选择 new 方法，创建对象指针。

* [C++创建对象的两种方法（C++用new和不用new创建类对象）](https://blog.csdn.net/lz20120808/article/details/40833517)

实例：

* [108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

## 溢出检查

TODO

## 语言特性

1
vector.size() 返回的数据类型非负，不能执行减法，

需要 int n = vector.size();

2
Effective STL no.21：总是让比较函数在等值情况下返回 false。

C++ 自定义排序 sort comp 方法，若两个元素相等需要返回 false，否则将导致访问越界，程序崩溃。

```C++
static bool cmp(const string &str1, const string &str2) {
    if (str1.size() != str2.size()) return str1.size() > str2.size();
    else {
        int n = str1.size();
        for (int i = 0; i < n; ++i) {
            if (str1[i] != str2[i]) return str1[i] > str2[i];
        }
        return false; // 两个字符串完全相等时返回 false 注意！！！
    }
}
```

3
涉及到最大最小值的边界判断时，不能直接将 int 类型的 INT_MIN INT_MAX，看作无穷小和无穷大，测试用例可能会卡极限。

如果测试输入为 int 类型的话，建议使用更大的

```C++
long val = LONG_MIN / LONG_MAX;
long long val = LLONG_MIN / LLONG_MAX;
```

作为边界条件进行比较。
TODO

## 参考链接
