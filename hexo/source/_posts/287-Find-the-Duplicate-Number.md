title: 287. Find the Duplicate Number
toc: true
date: 2016-07-05 21:10:32
tags:
- leetcode
- algorithm
categories:
---
[287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

## 思路1
该题目思路是与处理链表中带环位置相同的
[142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)
过程1 - 设置快慢两个指针，快指针一次走两步，慢指针一次走一步，在点a两个指针相遇
过程2 - 快指针从头开始，慢指针从A点开始，两个指针都一次走一步，则他们必在入环位置相遇

证明
>过程1 - 快指针走的路程为2s，慢指针走的路程为s，圆环长度为c，n为绕的圈数
则 s + n * c = 2s ==> s = n * c .....①
从开始到入环位置距离为x，相遇点A到入环位置为a
则 s = x + a ...②
结合①②得到 x + a = n * c ==> x = (n-1) * c + c-a
即过程2 就是 x = (n-1) * c + c-a的过程

在本题目中没有链表和指针，但是题目已经表示nums中的值都是[1 , n]的数字
因此可以模拟指针的过程

## 代码
```
public int findDuplicate(int[] nums) {
	int fast = nums[0];
	int slow = 0;
	while (fast != slow) {
		fast = nums[nums[fast]];
		slow = nums[slow];
	}
	fast = 0;
	while (fast != slow) {
		fast = nums[fast];
		slow = nums[slow];
	}
	return slow;
}
```

## 思路2
[二分](https://discuss.leetcode.com/topic/25580/two-solutions-with-explanation-o-nlog-n-and-o-n-time-o-1-space-without-changing-the-input-array/27)
找到中间数，然后计算数组中小于和大于中间数的个数，个数哪边大就说明那个方向有重复数字
继续遍历，缩小范围，直到找到该数字为止

```
public int findDuplicate1(int[] nums) {
	int s = 0;
	int e = nums.length - 1;
	while (s < e) {
		int cnt = 0;
		int m = (s + e) / 2;
		// 遍历所有的数字
		for (int i = 0; i < nums.length; i++) {
			if (nums[i] <= m) {
				cnt++;
			}
		}
		if (m < cnt) {
			e = m;
		} else {
			s = m + 1;
		}

	}
	return s;
}
```