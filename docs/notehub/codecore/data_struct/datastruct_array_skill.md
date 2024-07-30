---
title: 数组题目的解决技巧
date: 2023-08-21 22:24:39
tags:
  - 数据结构算法
---
# 双指针技巧
> - 在数组中没有真正意义上的指针，通常把索引当做数组中的指针

- 左右指针
   - 两个指针相向而行或者相背而行；
- 快慢指针
   - 两个指针同向而行，一快一慢；

## 技巧一、快慢指针
应用场景：

- 应用在【原地修改数组】类型的算法题目上
- 应用在【滑动窗口】类型的算法题目上

实践思路；

- 让慢指针走在后面，快指针走在前面探路，在根据题目中的要求，写出相关判断条件。
- 对于滑动窗口的类型而言：慢指针在后，快指针在前，两个指针中间的部分就是【窗口】，算法通过扩大或缩小窗口来解决相关问题。
- 框架Demo
```python

def demo(nums: List[int], val: int) -> int:
    fast, slow = 0, 0
    while fast < len(nums):
        if nums[fast] != val:
            nums[slow] = nums[fast]
            slow += 1
        fast += 1
    return slow
```
## 技巧二、左右指针
常用算法

- 二分查找

框架Demo
```python
def binarySearch(nums: List[int], target: int) -> int:
    # 一左一右两个指针相向而行
    left = 0
    right = len(nums)-1
    while left <= right:
        mid = (right + left) // 2
        if nums[mid] == target:
            return mid 
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```