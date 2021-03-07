---
title: 《算法导论》4.1 最大子数组章节的小问题
date: 2021-01-13 21:26:04
categories:
  - Algorithm & Programming
tags:
  - Introduction to Algorithms
---

老实说这是翻译校稿的锅，不过也的确有点难为他们了。

不过译者本身也脱不开干系。

<!--more-->

本章主要是将某股票购买问题化为最大子数组并用分治法解决：将已知的每日股价列出，选取其中合适的时间节点买入卖出以达到最大收益。

与简单的暴力求解不同，将价格数组`P`转为“价格波动数组`A`” -> 对于第 i 天的股票价格`P[i]`有`A[i] = P[i] - P[i-1]`。因此只需要找出 A 的非空连续**最大子数组**即可，即`SUM(A[i..j]) = MAX SUM(A[x..y])。

对于简单的“最低价格时买进，最高价格时卖出”策略，作者给了一个反例：
```
DAY	0	1	2	3	4
P	10	11	7	10	6
A		1	-4	3	-4
```

最优策略为在第二天买进，第三天卖出，收益为 3 美元。

以及一个完整的示例：

```
DAY	0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	16
P	100	113	110	85	105	102	86	63	81	101	94	106	101	79	94	90	97
A		13	-3	-25	20	-3	-16	-23	18	20	-7	12	-5	-22	15	-4	7
```

容易产生误解的部分就在这里：

> 最大子数组为 A[8..11]，其和为 43. 因此，你可以在第 8 天（7 天以后）买入股票，并在第 11 天后卖出，获得每股收益 43 美元。

参考代码如下（尽量还原书中伪码）：

```Go
func findMaxCrossingSubArray(s []int, low, mid, high int) (ansLeft, ansRight, ansSum int) {
	leftSum := ^(int(^uint(0) >> 1))
	sum := 0
	var maxLeft int
	for i := mid; i >= low; i-- {
		sum += s[i]
		if leftSum < sum {
			leftSum = sum
			maxLeft = i
		}
	}

	rightSum := ^(int(^uint(0) >> 1))
	sum = 0
	var maxRight int
	for i := mid + 1; i <= high; i++ {
		sum += s[i]
		if rightSum < sum {
			rightSum = sum
			maxRight = i
		}
	}

	return maxLeft, maxRight, leftSum + rightSum
}

func findMaximumSubArray(s []int, low, high int) (ansLeft, ansRight, ansSum int) {
	if high == low {
		return high, low, s[high]
	}
	mid := (low + high) / 2
	leftLow, leftHigh, leftSum := findMaximumSubArray(s, low, mid)
	rightLow, rightHigh, rightSum := findMaximumSubArray(s, mid+1, high)
	midLow, midHigh, midSum := findMaxCrossingSubArray(s, low, mid, high)
	if leftSum >= midSum && leftSum >= rightSum {
		return leftLow, leftHigh, leftSum
	}
	if midSum >= leftSum && midSum >= rightSum {
		return midLow, midHigh, midSum
	}
	return rightLow, rightHigh, rightSum
}
```

但实际上，如果不针对性的优化程序，这个程序对于上述两个例子的结果为：

```
2 2 3
7 10 43
```

如果说第二个结果还可以理解，但第一个例子的结果竟是一个单元素数组`A[2..2]`。

真正原因在于，书中的语言容易造成误解：`A[x..y]`为“在第 x 天买入股票，并在第 y 天后卖出。这行字如不咬文嚼字死抠字眼极易理解成“在第 x 天买入并在第 y 天卖出”。但实际上应为“在第 x 天买入并在第** y+1 **天卖出”。一个后字就让我抠了半个小时总觉得是程序逻辑问题……

因此第一个例子的结果`2 2 3`的正确解读为“在第二天 ($7) 买入并在第** 3 **天 ($10) 卖出”，盈利$3。正解。

本记录侧面表达出在算法中深刻理解每个变量被赋予意义的重要性。
