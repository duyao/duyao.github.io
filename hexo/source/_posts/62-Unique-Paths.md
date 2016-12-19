title: 62. Unique Paths
date: 2016-02-12 10:22:31
tags:
- leetcode
- algorithm
---

[62. Unique Paths](https://leetcode.com/problems/unique-paths/)

## 思路
不是用dfs或者bfs而是dp
`grid[i][j]`表示到达该位置的方法数目

```
public static int uniquePaths(int m, int n) {
	int grid [][] = new int[m][n];
	
	for(int i = 0; i < m; i++){
		for(int j = 0; j < n; j++){
			if(i == 0 || j == 0){
				grid[i][j] = 1;
			}else{
				grid[i][j] = grid[i-1][j] + grid[i][j-1];
			}
		}
	}
	return grid[m-1][n-1];
	
}
```