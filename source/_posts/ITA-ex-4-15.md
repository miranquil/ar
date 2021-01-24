---
title: 《算法导论》练习 4.1-5
date: 2021-01-14 23:16:12
urlname: ITA_ex_4_15
tags:
  - Introduction to Algorithms
categories:
  - Algorithm & Programming
---

> 使用如下思想为最大子数组问题设计一个非递归的、线性时间的算法。从数组的左边界开始，由左至右处理，记录到目前为止已经处理过的最大子数组。若已知 A[1..j] 的最大子数组，基于如下性质将解扩展为 A[i..j+1] 的最大子数组：A[1..j+1] 的最大子数组要么是 A[1..j] 的最大子数组，要么是某个子数组 A[i..j+1](1≦i≦j+1)。在已知 A[1..j] 的最大子数组的情况下，可以在线性时间内找出形如 A[i..j+1] 的最大子数组。

<!--more-->

首先明确了线性时间（复杂度以* n *记），分治等树结构不作考虑。题目给了足够的提示由左至右处理，唯一的问题在于如何寻找合适的`A[i..j+1]`。

对于一个已经找到最大子数组的子序列`A[1..j]`及下一个元素`A[j+1]`，要么无视新元素继续保持原结果，要么就是**新元素必须存在的情况下**从 i(1≦i≦j+1) 开始的新序列（其中包含 i=j+1 时形成的单元素序列）。

将上文中“合适的`A[i..j+1]`”记作`s[j+1]`。对于这个序列而言，由于新元素`A[j+1]`必须存在，因此无需考虑新元素本身，只需考虑如何求得`s[j]`即可。

## 推导

从简单的小数组开始。

* i=0 时，`s[0]`必为`[ A[0] ]`的单元素数组。
* i=1 时，由于`A[1]`必须包含，则`s[1]`要么为`[ A[0], A[1] ]`，要么为仅包含`A[1]`的单元素数组 => 即`max(s[0], s[0] + A[1])`。
* i=2 时，`s[2]`要么为`[ s[1], A[2] ]`，要么为仅包含`A[2]`的单元素数组。

猜测： `s[n] = max(s[n-1]+A[n], A[n])`

* i=n 时，`s[n]`要么为`[ s[n-1], A[n] ]`，要么为仅包含`A[n]`的单元素数组。

同样成立。

i=0 和 n 的情况分别构成了循环不变式，因此思路的正确性可行。

于此`s[n]`已经可以求出，并且无需额外迭代步骤。

## Code

参考代码：

```Go
func maxSubArray(nums []int) (left, right, sum int) {
	maxSum := nums[0]
	var s int
	var maxLeft, maxRight, tmpLeft int

	for i, n := range nums {
		// initial situation: maxSubArray of a single array is itself
		if i == 0 {
			s = n
			maxLeft, maxRight, tmpLeft = 0, 0, 0
			continue
		}

		// "s" or s[j] = max(maxSubArray(nums[i..j]) = max(s[j-1]+nums[j], nums[j])
		// so if s[j-1] < 0, s[j] = nums[j]
		if s < 0 {
			s = n
			tmpLeft = i
		} else {
			s += n
		}

		// maxSubArray(nums[1..j+1] = max(s[j] + nums[j+1], maxSubArray(nums[1..j])
		// we already have maxSum=maxSubArray(nums[1..j])
		if s > maxSum {
			maxSum = s
			maxLeft = tmpLeft
			maxRight = i
		}
	}

	return maxLeft, maxRight, maxSum
}
```
