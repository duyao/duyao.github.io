title: 221. Maximal Square
toc: true
date: 2016-04-15 15:17:00
tags:
- leetcode
- algorithm
categories:
---
[221. Maximal Square](https://leetcode.com/problems/maximal-square/)


![dp的值选择](http://7xilc8.com1.z0.glb.clouddn.com/221.%20Maximal%20Square.png)

```
public int maximalSquare(char[][] matrix) {
	if (matrix == null || matrix.length == 0) {
		return 0;
	}
	//dp[i][j]表示以ij为右下角的含1正方型的最大边长
	int res = matrix[0][0] - 48;
	int[][] dp = new int[matrix.length][matrix[0].length];
	for (int i = 0; i < matrix.length; i++) {
		dp[i][0] = matrix[i][0] - 48;
		res = Math.max(res, dp[i][0]);
	}
	for (int i = 0; i < matrix[0].length; i++) {
		dp[0][i] = matrix[0][i] - 48;
		res = Math.max(res, dp[0][i]);
	}
	for (int i = 1; i < matrix.length; i++) {
		for (int j = 1; j < matrix[0].length; j++) {
			if (matrix[i][j] == '0') {
				dp[i][j] = 0;
			} else {
				//tmp是最该的覆盖区域
				int tmp = Math.min(dp[i - 1][j], dp[i][j - 1]);
				//matrix[i - tmp][j - tmp]-48表示该形状左上角的数字
				dp[i][j] = tmp + matrix[i - tmp][j - tmp]-48;

			}
			res = Math.max(res, dp[i][j]);
		}
	}
	return res*res;
}
```