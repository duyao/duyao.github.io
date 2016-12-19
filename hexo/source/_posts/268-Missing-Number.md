title: 268. Missing Number
toc: true
date: 2016-07-05 13:11:04
tags:
- leetcode
- algorithm
categories:
---
[268. Missing Number](https://leetcode.com/problems/missing-number/)

## 方法一
### 思路
利用[0, n]的区间内和是`(0 + n) * (n + 1) / 2`
因为缺少一个数，那么和减去数组中的所有数字就是剩下的

## 方法二
### 思路
利用`a ^ b ^b = a`
由于`nums[i] = i`因此最后剩下的就是缺少的数字
例如
k = k ^ i ^ nums[i]
i = {0,1,2,3}
nums = {0,2,3,4}
k = 4
最后只剩下1没有消除
因此缺少1

与改题目类似的还有[136. Single Number](https://leetcode.com/problems/single-number/)
`a ^ b ^b = a`

### 代码
```
public int missingNumber(int[] nums) {
	int k = nums.length;
	for (int i = 0; i < nums.length; i++) {
		// 可能会溢出
		// k = k ^ i ^ nums[i];
		k ^= i;
		k ^= nums[i];
	}
	System.out.println(k);
	return k;
}
```
