title: 363. Max Sum of Rectangle No Larger Than K
toc: true
date: 2016-06-27 21:22:49
tags: 
- leetcode
- algorithm
categories:
---
[363. Max Sum of Rectangle No Larger Than K](https://leetcode.com/problems/max-sum-of-sub-matrix-no-larger-than-k/)

## Max Sum of Rectangle

### 思路

[Max Sum of Rectangle](https://www.youtube.com/watch?v=yCQN096CwWM&noredirect=1)

通过控制左右来遍历区间列的和，对于每个列的和，找到最大值

### 代码
```
//找出最大的
public int maxSumSubmatrix0(int[][] matrix, int k) {

	int maxSum = Integer.MIN_VALUE;
	int up = 0;
	int down = 0;
	int right = 0;
	int left = 0;
	for (int i = 0; i < matrix[0].length; i++) {
		int a[] = new int[matrix.length];
		for (int j = i; j < matrix[0].length; j++) {
			// 填充a
			for (int h = 0; h < matrix.length; h++) {
				a[h] += matrix[h][j];
			}
			// 找到该列最大值
			int tmpSum = 0;
			int s = 0;
			int e = 0;
			int dp[] = new int[a.length];
			dp[0] = a[0] > 0 ? a[0] : 0;
			for (int h = 1; h < a.length; h++) {
				dp[h] = dp[h-1] + a[h] > 0 ? dp[h-1] + a[h] : 0;
				
				if (dp[h] > tmpSum) {
					tmpSum = dp[h];
					if (dp[h - 1] > 0) {
						e = h;
					} else {
						s = h;
					}
				}
			}
			if (tmpSum > maxSum) {
				maxSum = tmpSum;
				up = s;
				down = e;
				left = i;
				right = j;
			}

		}
	}
	System.out.println("up"+up+"down"+down+"left"+left+"right"+right);
	return maxSum;
}
```

## Max Sum of Rectangle No Larger Than K

### 思路
与前面的方法相似，对于处理列和有所变化
这里使用set来记录所有子串的和(从0到所有位置),sum表示当前从0到该位置的和
a + b = sum
a是set中记录的值，b是我们所有的值，题目中要求 b <= k
那么 sum - a <= k , a >= sum - k
那么只要找到set中最小的sum-k即可，这就是treeset中的ceiling方法

> Integer java.util.TreeSet.ceiling(Integer e)
Returns the least element in this set greater than or equal to the given element, or null if there is no such element.

### 代码
```
//最大且小于k
public int maxSumSubmatrix(int[][] matrix, int k) {

	int maxSum = Integer.MIN_VALUE;
	int up = 0;
	int down = 0;
	int right = 0;
	int left = 0;
	for (int i = 0; i < matrix[0].length; i++) {
		int a[] = new int[matrix.length];
		for (int j = i; j < matrix[0].length; j++) {
			// 填充a
			for (int h = 0; h < matrix.length; h++) {
				a[h] += matrix[h][j];
				
			}
			
			// 找到最大值
			int tmpSum = 0;
			TreeSet<Integer> set = new TreeSet<Integer>();
			//结果是本身，比如a=[1,2],k=1
			set.add(0);
			for (Integer integer : a) {
				tmpSum += integer;
				Integer tmp = set.ceiling(tmpSum - k);
				if(tmp != null){
					maxSum = Math.max(maxSum, tmpSum - tmp);
				}
				set.add(tmpSum);
				
			}
			
			

		}
	}
	return maxSum;
}
```


## Smallest subarray with sum greater than a given value
[greater than a given value](http://www.geeksforgeeks.org/minimum-length-subarray-sum-greater-given-value/)

