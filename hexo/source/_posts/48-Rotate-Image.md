title: 48. Rotate Image
toc: true
date: 2016-05-04 21:47:43
tags: leetcode
categories: algorithm
---
[48. Rotate Image](https://leetcode.com/problems/rotate-image/)

## 思路

### clockwise rotate

- first reverse up to down
- then swap the symmetry

```
1 2 3     7 8 9     7 4 1
4 5 6  => 4 5 6  => 8 5 2
7 8 9     1 2 3     9 6 3
```

### anticlockwise rotate
- first reverse left to right
-  then swap the symmetry

```
1 2 3     3 2 1     3 6 9
4 5 6  => 6 5 4  => 2 5 8
7 8 9     9 8 7     1 4 7
```

## 代码

```
// clockwise
// 1.up to down
// 2. swap symmetry
// anticlockwise
// 1.left to right
// 2.swap symmetry
public void rotate(int[][] matrix) {
	if (matrix == null || matrix.length == 0) {
		return;
	}
	// 1.up to down
	for (int i = 0; i < matrix.length / 2; i++) {
		for (int j = 0; j < matrix.length; j++) {
			int k = matrix.length - 1 - i;
			int tmp = matrix[i][j];
			matrix[i][j] = matrix[k][j];
			matrix[k][j] = tmp;
		}

	}
	// 2. swap symmetry
	for (int i = 0; i < matrix.length; i++) {
		for (int j = i + 1; j < matrix.length; j++) {
			int tmp = matrix[i][j];
			matrix[i][j] = matrix[j][i];
			matrix[j][i] = tmp;
		}
	}

}

```
