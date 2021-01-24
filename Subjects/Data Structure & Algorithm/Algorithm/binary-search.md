# 二分查找

二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法。但是，折半查找要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。

目录：

- [二分查找](#二分查找)
  - [查找过程](#查找过程)
  - [算法要求](#算法要求)
  - [优缺点分析](#优缺点分析)
  - [复杂度分析](#复杂度分析)
  - [代码模板](#代码模板)
    - [基本框架](#基本框架)
    - [模板 Ⅰ](#模板-ⅰ)
    - [模板 Ⅱ](#模板-ⅱ)
    - [模板 Ⅲ](#模板-ⅲ)
  - [参考链接](#参考链接)

## 查找过程

首先，假设表中元素是按升序排列，将表中间位置记录的关键字与查找关键字比较，如果两者相等，则查找成功；否则利用中间位置记录将表分成前、后两个子表，如果中间位置记录的关键字大于查找关键字，则进一步查找前一子表，否则进一步查找后一子表。重复以上过程，直到找到满足条件的记录，使查找成功，或直到子表不存在为止，此时查找不成功。

## 算法要求

1. 必须采用顺序存储结构 [^顺序存储结构]。（补充：基于链表的 [跳表](https://baike.baidu.com/item/%E8%B7%B3%E8%A1%A8/22819833) 也能实现二分查找）
[^顺序存储结构]:顺序存储结构是存储结构类型中的一种，该结构是把逻辑上相邻的结点存储在物理位置上相邻的存储单元中，结点之间的逻辑关系由存储单元的邻接关系来体现。通常顺序存储结构由数组来描述。该存储结构特点，优点：随机存取表中元素、储存密度大。缺点：插入和删除操作需要移动元素。

2. 必须按关键字大小有序排列。

## 优缺点分析

* 优点：比较次数少，查找速度快，平均性能好。
* 缺点：要求待查表为有序表，且插入删除困难。

故：二分查找方法适用于不经常变动而查找频繁的有序列表。

## 复杂度分析

* 时间复杂度：O(logN)。
* 空间复杂度：O(1)。

## 代码模板

### 基本框架

```java
int binarySearch(int[] nums, int target) {
    if(nums == null || nums.length == 0) return -1;

    int left = 0, right = ..., mid;

    while(...) {
        mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            ...
        } else if (nums[mid] < target) {
            left = ...
        } else if (nums[mid] > target) {
            right = ...
        }
    }
    return ...;
}
```

注意：

* 使用二分查找时，避免使用 `else`，而是将所有情况都用 `else if` 说明清楚，以便展现所有细节并分析。
* `...` 标记部分，为着重注意的细节部分，该部分可能随着不同的题目要求而变化。
* 为了防止计算 `mid` 时溢出，代码中用 `left + (right - left) / 2` 来替代 `(left + right) / 2`，两者结果相同，但有效避免了 `left` 和 `right` 相加太大而溢出。

### 模板 Ⅰ

适用场景：寻找一个数。
即搜索一个数，若该数存在，返回其索引，否则返回 -1。

**eg 1. 左闭右闭**

```java
int binarySearch(int[] nums, int target) {
    if(nums == null || nums.length == 0) return -1;

    int left = 0, right = nums.length - 1, mid; // 注意

    while(left <= right) {   // 注意
        mid = left + (right - left) / 2; 
        if(nums[mid] == target)
            return mid; 
        else if (nums[mid] < target)
            left = mid + 1;  // 注意
        else if (nums[mid] > target)
            right = mid - 1; // 注意
    }

    return -1;
}
```

**注意：**

1\. `while` 循环条件用 `<=`。

由于初始化 `right` 的赋值是 `nums.length - 1`，即最后一个元素的索引，而不是 `nums.length`。前者相当于左闭右闭区间 [left, right]，后者相当于左闭右开区间 [left, right)。当使用两端闭区间 [left, right] 作为搜索区间的时候，while 循环应该搜索 **包括边界在内** 的该区间中的所有数字，即当 **搜索区间为空的时候终止**。而由于两端都为闭区间，若想要搜索区间为空，则需要 **左端索引大于右边索引**，如：[3, 2]。

`while (left <= right)` 的终止条件等价为 `left == right + 1`，对应区间为 [right + 1, right]，该 **区间为空**，表示该循环正确的搜索了整个区间，故此时 `while` 循环终止是正确的，不需要做额外处理。而 `while (left < right)` 的终止条件等价为 `left == right`，对应区间为 [left, right]，该 **区间非空**，故此时退出 `while` 循环还需要对 `nums[left]` 的值进行额外处理。

```java
//...
while(left < right) {
    // ...
}
return nums[left] == target ? left : -1;
```

综上，从代码简洁性考虑，当 `right` 的赋值是 `nums.length - 1` 时，`while` 循环条件应使用 `<=`。

2\. `left = mid + 1`，`right = mid - 1`

从搜索区间考虑，由于已经判断了 `mid` 不是要找的 target，故需要将 `mid` 从搜索区间中去除，所以应该使用 `left = mid + 1` 或 `right = mid - 1` 将搜索区间变为 [mid+1, right] 或 [left, mid-1]。

**思维过程（左闭右闭）：**

1. 首先选择初始化 `right = nums.length - 1` 决定了搜索区间为闭区间 [left, right]；
2. 因为搜索区间为闭区间，所以决定了 `while (left <= right)`（以保证退出循环的时候区间内所有的数都被扫描到）；
3. 同时决定了 `left = right + 1` 和 `right = mid - 1`（以保证 `mid` 被排除掉区间）；
4. 因为只需要找到一个 `target` 目标，故当 `nums[mid] == target` 时，可以立即返回。

**递归实现（左闭右闭）：**

```java
// 左闭右闭 递归写法
class Solution {
    public int search(int[] nums, int target) {
        if (nums == null && nums.length == 0) return -1;

        int mid = (nums.length - 1) / 2;

        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) return binarySearch(nums, target, mid + 1, nums.length - 1);
        else return binarySearch(nums, target, 0, mid - 1);
    }

    private int binarySearch(int[] nums, int target, int left, int right) {
        if (left > right) return -1;

        int mid = left + (right - left) / 2;

        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) return binarySearch(nums, target, mid + 1, right);
        else return binarySearch(nums, target, left, mid - 1);
    }
}
```

**eg 2. 左闭右开**

```java
int binarySearch(int[] nums, int target) {
    if(nums == null || nums.length == 0) return -1;

    int left = 0, right = nums.length, mid; // 注意

    while (left < right) {  // 注意
        mid = left + (right - left) / 2;
        if (nums[mid] == target) 
            return mid;
        else if (nums[mid] < target) 
            left = mid + 1; // 注意
        else if (nums[mid] > target) 
            right = mid;    // 注意
    }
    
    return -1;
}
```

**注意：**

1\. `while` 循环条件用 `<`。

与 eg 1. 左闭右闭对比可知。由于初始化 `right` 的赋值是 `nums.length`，相当于左闭右开区间 [left, right)。当使用左闭右开区间 [left, right) 作为搜索区间的时候，若想要搜索区间为空，只需要 **左端索引等于右边索引** 即可，如：[2, 2)。

`while (left < right)` 的终止条件等价为 `left == right`，对应区间为 [left, left)，该 **区间为空**，故此时退出 `while` 循环是正确的，不需要做额外处理。

2\. `left = mid + 1`，`right = mid`

从搜索区间考虑，由于右边界为开区间，故将 `right` 赋值为 `mid`，搜索区间变为 [left, mid)，即可将 `mid` 从搜索区间中去除。

**思维过程（左闭右开）：**

1. 首先选择初始化 `right = nums.length` 决定了搜索区间为左闭右开 [left, right)；
2. 因为搜索区间为闭区间，所以决定了 `while (left < right)`（以保证退出循环的时候区间内所有的数都被扫描到且不会出现重复与死循环）；
3. 同时决定了 `left = right + 1` 和 `right = mid`（因为右端为开区间，故 `right` 不需要 -1）；
4. 因为只需要找到一个 `target` 目标，故当 `nums[mid] == target` 时，可以立即返回。

**递归实现（左闭右开）：**

```java
// 左闭右开 递归写法
class Solution {
    public int search(int[] nums, int target) {
        if (nums == null && nums.length == 0) return -1;

        int mid = nums.length / 2;

        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) return binarySearch(nums, target, mid + 1, nums.length);
        else return binarySearch(nums, target, 0, mid);
    }

    private int binarySearch(int[] nums, int target, int left, int right) {
        if (left >= right) return -1;

        int mid = left + (right - left) / 2;

        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) return binarySearch(nums, target, mid + 1, right);
        else return binarySearch(nums, target, left, mid);
    }
}
```

**模板 Ⅰ 算法局限性**

当数组有重复数据如：`nums = [1,2,2,2,3]` 并要求给出 target 为 2 的最左侧或最右侧索引时，该算法无法处理（若采用找到其中一个数字，并向左或右线性查找的方法，则无法保证对数级时间复杂度）。

典型例题：[704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

### 模板 Ⅱ

适用场景：寻找左侧边界
即搜索一个数，若有多个重复的数，则返回最左侧的边界的索引，否则返回 -1。

**eg 1. 左闭右闭**

```java
int left_bound(int[] nums, int target) {
    if(nums == null || nums.length == 0) return -1;

    int left = 0, right = nums.length - 1, mid;

    // 搜索区间为 [left, right]
    while (left <= right) {
        mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            right = mid - 1;  // 收缩右侧边界
        } else if (nums[mid] < target) {
            left = mid + 1;   // 搜索区间变为 [mid+1, right]
        } else if (nums[mid] > target) {
            right = mid - 1;  // 搜索区间变为 [left, mid-1]
        } 
    }

    // 检查越界情况
    if (left == nums.length || nums[left] != target) return -1;

    return left;
}
```

**注意：**

1\. 对 nums[mid] == target 情况进行处理

当搜索到 `target` 时，不立刻返回，而是缩小搜索区间的上界 `right`，在 [left, mid - 1] 中继续搜索，即不断向左收缩，直到最后 `left` 锁定左侧边界。

```java
if (nums[mid] == target) {
    right = mid - 1;
}
```

2\. 返回 left

由于 `nums[mid] == target` 时，`right` 指针变为 `mid - 1`，向左收缩，将会错过左边界，而退出循环的时候，因为 `left = right + 1`，`left` 将指向左边界，应返回 `left`。

3\. 越界处理

由于最后由 `left` 指针锁定左侧边界，故在判断 `nums[left]` 值是否等于 `target` 之前，需要先判断其是否越界。

**思维过程（左闭右闭）：**

1. 首先选择初始化 `right = nums.length - 1`，决定了搜索区间是 [left, right]；
2. 同时决定了 `while (left <= right)`；
3. 同时也决定了 `left = mid + 1` 和 `right = mid - 1`；
4. 因为我们需找到 target 的最左侧索引，所以当 `nums[mid] == target` 时不要立即返回，而要收紧右侧边界以锁定左侧边界，而因为是闭区间故收缩区间的时候也需要 -1；
5. 而因为通过 `left` 指针锁定左侧边界，而 `left` 取值可能为 `nums.length`，故判断 `nums[left] == target` 之前需要先判断其是否溢出。

**eg 2. 左闭右开**

```java
int left_bound(int[] nums, int target) {
    if(nums == null || nums.length == 0) return -1;

    int left = 0, right = nums.length, mid; // 注意
    
    while (left < right) { // 注意
        mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            right = mid;   // 注意
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;   // 注意
        }
    }

    // 检查越界情况
    if (left == nums.length || nums[left] != target) return -1;
    // if (right == nums.length || nums[right] != target) return -1;

    return left;
    // return right;
}
```

**注意：**

1\. 对 nums[mid] == target 情况进行处理

当搜索到 `target` 时，不立刻返回，而是缩小搜索区间的上界 `right`，在 [left, mid) 中继续搜索，即不断向左收缩，直到最后 `left`、`right` 锁定左侧边界。

```java
if (nums[mid] == target) {
    right = mid;
}
```

2\. 返回 left 或 right

由于退出 while 循环时，`left == right`，故返回两者均可。

3\. 越界处理

根据返回值 `left` 或 `right`，在判断其值是否为 `target` 前，需要先判断其是否越界。

**思维过程（左闭右开）：**

1. 首先选择初始化 `right = nums.length`，决定了搜索区间是 [left, right)；
2. 同时决定了 `while (left < right)`；
3. 同时也决定了 `left = mid + 1` 和 `right = mid`；
4. 因为我们需找到 `target` 的最左侧索引，所以当 `nums[mid] == target` 时不要立即返回，而要收紧右侧边界以锁定左侧边界；
5. 而因为通过 `left` 指针锁定左侧边界，而 `left` 取值可能为 `nums.length`，故判断 `nums[left] == target` 之前需要先判断其是否溢出。

典型例题：[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

### 模板 Ⅲ

适用场景：寻找右侧边界
即搜索一个数，若有多个重复的数，则返回最右侧的边界的索引，否则返回 -1。

**eg 1. 左闭右闭**

```java
int right_bound(int[] nums, int target) {
    if(nums == null || nums.length == 0) return -1;

    int left = 0, right = nums.length - 1, mid;

    while (left <= right) {
        mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            left = mid + 1;  // 这里改成收缩左侧边界即可
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    
    // 与左边界相反，这里由 right 指针锁定右边界，故改为检查 right 指针越界的情况
    if (right == -1 || nums[right] != target) return -1;

    return right;
}
```

**eg 2. 左闭右开**

```java
int right_bound(int[] nums, int target) {
    if(nums == null || nums.length == 0) return -1;

    int left = 0, right = nums.length, mid;
    
    while (left < right) {
        mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            left = mid + 1;  // 注意
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;
        }
    }
    
    if (left == 0 || nums[left - 1] != target) return -1;
    // if (right == 0 || nums[right - 1] != target) return -1;

    return left - 1; // 注意
    // return right - 1;
}
```

**注意：**

1\. 最后求出的索引需要 -1。

由于对 `left` 的更新是 `left = mid + 1`，即当循环结束的时候，`nums[left]` 一定不等于 `target`，而 `nums[left - 1]` 可能等于 `target`。

**思维过程：**

与 模板 Ⅱ类似，只需将对应的 `nums[left] == target` 内容改为向右侧收缩即可（即移动 `left` 指针 `left = mid + 1`）。

典型例题：[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

## 参考链接

* [二分查找_百度百科](https://baike.baidu.com/item/%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE)
* [二分查找](https://www.jianshu.com/p/a3e5372053eb)
* [二分查找细节详解，顺便赋诗一首](https://leetcode-cn.com/problems/binary-search/solution/er-fen-cha-zhao-xiang-jie-by-labuladong/)
* [二分查找 - LeetBook](https://leetcode-cn.com/leetbook/detail/binary-search/)
