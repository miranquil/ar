---
title: 1. The Sum
urlname: leetcode_the_sum
date: 2021-01-12 22:12:50
tags:
  - LeetCode
photos:
  - /leetcode.png
---

<!--more-->

> Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.
>
> You may assume that each input would have exactly one solution, and you may not use the same element twice.
>
> You can return the answer in any order.

## 穷举

暴力枚举可解，二重循环，对每个元素Ni扫描整个数组检查`if Ni+Nj == target and i!=j`，时间复杂度O(n²)。

虽然空间复杂度仅为O(n)但显然不是想要的。

## 优化

记忆化搜索，对于每个输入元素Ni，使用哈希表m记录其与target的差值。

则一旦该值存在于输入序列，Ni与target-Ni即为所需的两个数。

此法时空复杂度均为O(n)。

循环不变式证明：

* *假定存在表m满足m[n] = target-n，n∊S*
* **初始化**：迭代前，m=∅
* **保持**：若对于i(i>=1 AND i<S.LENGTH)，有m[Si] == target-Si，则m[S(i+1)] == target-S(i+1)。 由于m[n]为手动计算，因此保证结果成立。
* **终止**：迭代结束后，对任意n∊S均有m[n] == target-n。

## 伪码（Python）

```Python
for n in nums:
    if m[target-n]:
        return n, target-n
    m[target-n] = n
```

## ANS

考虑到题目要的结果是对应的序号，因此有少许变动。

*优化：*对于输入的数组，从左到右迭代，则最终答案`{x, y}`必定满足m[y]会在迭代到nums[y]之前生成（迭代到nums[x]时）。因此无需存储m[x]的值。

```Go
func twoSum(nums []int, target int) []int {
	m := make(map[int]int)
	for i, n := range nums {
		cut, ok := m[n]
		if ok {
			return []int{cut, i}
		}
		m[target-n] = i
	}
	return nil
}
```

> Runtime: 4 ms, faster than 93.39% of Go online submissions for Two Sum.
> Memory Usage: 3.2 MB, less than 99.88% of Go online submissions for Two Sum.