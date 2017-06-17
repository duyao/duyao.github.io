title: 73. Set Matrix Zeroes
toc: true
date: 2016-07-16 20:41:29
tags:
- leetcode
- algorithm
categories:
---
# 原地重置问题

根据特定的条件将数组变化，不用额外的空间
[73. Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/)
[289. Game of Life](https://leetcode.com/problems/game-of-life/)

## 73. Set Matrix Zeroes
[73. Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/)
该题目是用第一行和第一列记录状态，然后其余部分根据第一行和第一列来改变
1. 设置row和col来标记第一行和第一列是否需要改变
2. 遍历其余部分，该位置为0，那么将该位置对应的第一行和第一列的值设置为0
3. 根据第一行和第一列，只有其中有1个为0，就设置其值为0
4. 根据第一步的row和col，来改变第一行和第一列

```
public void setZeroes(int[][] matrix) {
	int m = matrix.length;
	int n = matrix[0].length;
	// 设置row和col
	boolean col = false;
	boolean row = false;
	for (int i = 0; i < m; i++) {
		if (matrix[0][i] == 0) {
			col = true;
			break;
		}
	}
	for (int i = 0; i < n; i++) {
		if (matrix[i][0] == 0) {
			row = true;
			break;
		}
	}
	// 根据数组设置第一行第一列
	for (int i = 1; i < m; i++) {
		for (int j = 1; j < n; j++) {
			if (matrix[i][j] == 0) {
				matrix[i][0] = 0;
				matrix[0][j] = 0;
			}
		}
	}
	// 根据第一行和第一列设置数组
	for (int i = 1; i < m; i++) {
		for (int j = 1; j < n; j++) {
			if (matrix[i][0] == 0 || matrix[0][j] == 0) {
				matrix[i][j] = 0;
			}
		}
	}
	// 设置第一行和第一列
	if (row) {
		for (int i = 0; i < m; i++) {
			matrix[i][0] = 0;
		}
	}
	if (col) {
		for (int i = 0; i < n; i++) {
			matrix[0][1] = 0;
		}
	}

}
```

## 289. Game of Life
[289. Game of Life](https://leetcode.com/problems/game-of-life/)
利用变色法来记录状态
1->0 == 2
0->1 == -1
统计个数是查看1(原来和现在都是1)和2(原来是1现在是0)个数
先更新成带变色的数组，然后再还原

```
public void gameOfLife(int[][] board) {
	//1->0 == 2
	//0->1 == -1
	//使用假色，1的个数=大于零的个数，因为可能有1->0 == 2
	for(int i = 0; i < board.length; i++){
		for(int j = 0; j < board[0].length; j++){
			int k = f(board, i , j);
			if(board[i][j] == 1 && (k < 2 || k > 3)){
				//1->0 == 2
				board[i][j] = 2;
			}else if(board[i][j] == 0 && k == 3){
				//0->1 == -1
				board[i][j] = -1;
			}
		}
	}
	//恢复
	for(int i = 0; i < board.length; i++){
		for(int j = 0; j < board[0].length; j++){
			if(board[i][j] == 2){
				//1->0 == 2
				board[i][j] = 0;
			}else if(board[i][j] == -1){
				//0->1 == -1
				board[i][j] = 1;
			}
		}
	}
	

}
//计算个数
public int f(int[][] x, int a, int b){
	int cnt = 0;
	int[] k = {-1, 0, 1};
	for(int i = 0; i < 3; i++){
		int curi = a + k[i];
		for(int j = 0; j < 3; j++){
			int curj = b + k[j];
			if(curi >= 0 && curj >= 0 && curi < x.length && curj < x[0].length 
					&& x[curi][curj] > 0 && !(k[i] == 0 && k[j] == 0)){
				cnt++;
			}
		}
	}
	return cnt;
}
```