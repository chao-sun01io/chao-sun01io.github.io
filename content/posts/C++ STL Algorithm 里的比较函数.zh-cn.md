---
aliases: []
tags: []
publish: true
title: C++ STL Algorithm 里的比较函数
date: 2026-05-10T12:15:37Z
lastmod: 2026-05-11T20:58:57Z
category: posts
tags: [C++, STL]
co-author: Doubao AI, Gemini
---

这篇我想聊的是，类似 `std::sort`、`std::priority_queue` 或者 `lower_bound/upper_bound` 的定制比较函数。

```cpp
template< class RandomIt, class Compare > void sort( RandomIt first, RandomIt last, Compare comp );

template< class ForwardIt, class T, class Compare >
ForwardIt lower_bound( ForwardIt first, ForwardIt last, const T& value, Compare comp );

template< class ForwardIt, class T, class Compare >
ForwardIt upper_bound( ForwardIt first, ForwardIt last, const T& value, Compare comp );

template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type>
> class priority_queue;
```

在面试做题或者实际工作中，有时候我们遇到的类型 T 是一个复合类型，无法应用默认的比较函数；或者我们对于排序有自己的规则，这种情况就需要实现定制化的比较函数，通常通过仿函数（Functor）或者 Lambda 表达式实现。

在面试无法查文档的情况下，如何搞清楚 comp 参数的规则，从而正确地实现需求（例如从大到小排序、小根堆等）？我们可以将其分为以下几类：

## 一、Sort 排序类：“谁优先”/ Who comes first

对于如下比较函数：

```cpp
bool comp(const T& a, const T& b) { return a < b; }
```

如果 comp 函数返回 true，则意味着 a 会放到 b 前面。因此，默认函数 `std::less<T>()` 为 true 时，a 会在 b 前面——即升序排列；`std::greater<T>()` 为 true 时，a 大于 b 时 a 在 b 前面——即降序排列。

## 二、Priority Queue: “谁更不重要” / who is less important

这是最常见的混淆点。在 std::priority_queue 中，函数对象定义了哪个元素具有较低优先级（因此会被推到底层堆的后面）。

对于如下比较函数：

```cpp
bool comp(const T& a, const T& b) { return a < b; }
```

comp 函数返回 true 时，表示 a 更不重要，所以 a 会放到堆的更底层，高优先级的元素则留在堆顶；因此，默认的 less 函数定义的是大根堆。

如果想实现小根堆，可以使用 `std::greater`，示例如下：

```cpp
std::priority_queue<int, std::vector<int>, std::greater<int>>
        min_heap;
```

## 三、`std::lower_bound / std::upper_bound` 分区逻辑

这些二分查找算法在已排序的数组、容器中查找“分区点”，其 comp 参数的逻辑与排序、优先队列有所不同，核心是判断元素与目标值的分区关系。

### 1. lower_bound

Returns the first element `e` where `comp(e, value)` is **false** (the first element not less than `value`).

```cpp
lower_bound(a.begin(), a.end(), target,
            [](int element, int target) { return element < target; });
```

lower_bound 对已排序数组应用 lambda 表达式时，会返回数组中第一个使得 lambda 表达式为 false 的元素的迭代器，本质上就是查找第一个大于等于 target 的元素。

注意 lower_bound 的 `comp(element, value)` 与 sort 的 `comp(a, b)` 语义一致——返回 true 都意味着“前者排在后者之前”。这也是 lower_bound 能与 sort 共用同一个比较器的根本原因。

### 2. upper_bound

Returns the first element `e` where `comp(value, e)` is **true** (the first element strictly greater than `value`).

```cpp
upper_bound(a.begin(), a.end(), target,
            [](int target, int element) { return target < element; });
```

upper_bound 会查找第一个使得 comp 函数为 true 的元素，本质上就是查找第一个严格大于 target 的元素。

>[!NOTE]
>For `upper_bound`, the order of arguments is flipped: it checks `comp(value, element)` rather than `comp(element, value)`

## 四、Summary Comparison Table

|Tool|Functor Meaning|Result of `comp(a, b) == true`|
|---|---|---|
|`std::sort`|Order|`a` is placed before `b`|
|`std::priority_queue`|Weakness|`a` is less important; `b` moves toward `top()`|
|`std::lower_bound`|Search|`a` is still "before" the target value|
|`std::upper_bound` | Search | `a` (the value) is "before" element `b`|
    
小技巧：
1.  先决定前后：在 sort, priority_queue 和 lower_bound 中， 函数参数顺序和函数体实现时一致（比如 a 在前， b 在后），可以避免混淆; 而 upper_bound 中， 函数参数顺序和函数体实现时也是一致的，但是target 在前， element 在后，这时就要注意区分了。
2. 再决定比较符号： 应用上表， 记住返回 true 时代表的含义。

## 五、进阶

1. C++ 14 Transparent Comparison
2. C++ 20: Ranges and projection comparison
