title: 130. Surrounded Regions
toc: true
date: 2016-03-26 19:52:13
tags: leetcode
categories: algorithm
---

[130. Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)

## 思路

是图的遍历，类似于岛屿的个数
但是边缘的全部不能被该成X
因此可以先遍历所有边缘的O，即把本身和与其相连的O全部变为b

然后再顺序遍历，将b变为O，将O变为X

**注意**
bfs遍历时，访问元素可以是入队也可以是出队

- 入队

如果不设置入队判断，必须是入队访问，即入队是将O变为b
因为元素可能会重复入内

- 出队

出队访问的话，有可能会导致元素重复入队，造成死循环，因此设置是否入队标志

## 代码

```
static class Node{
	int x;
	int y;
	//记录坐标信息，不需要设置访问值，因为被访问后的值会被改变
	Node(int x, int y){
		this.x = x;
		this.y = y;
	}
}
//只访问边界的坐标，将边界坐标o置为b，且其相邻的o也遍历置为b
public void bfsBoundary(char[][] c, int x, int y,boolean[][] in){
	int len = c[0].length;
	int wid = c.length;
	int[] xx = {0, 1, 0, -1};
	int[] yy = {-1, 0, 1, 0};

	LinkedList<Node> queue = new LinkedList<Node>();
	queue.add(new Node(x, y));
	c[x][y] = 'b';
//	in[x][y] =true;
	while(!queue.isEmpty()){
		Node cur = queue.poll();
//		c[cur.x][cur.y] = 'b';
		for(int i = 0; i < xx.length; i++){
			int curX = cur.x + xx[i];
			int curY = cur.y + yy[i];
//			&& !in[curX][curY]
			if( curX >= 0 && curX < wid && curY >= 0 && curY < len && c[curX][curY] == 'O' ){
				queue.add(new Node(curX, curY));
//				in[curX][curY] = true;
				c[curX][curY] = 'b';
			}
		}
	}

}
public void solve(char[][] board) {

	if(board == null || board.length == 0){
		return;
	}

	boolean[][] in = new boolean[board.length][board[0].length];
	int len = board[0].length;
	int wid = board.length;
	//访问边缘
	for(int i = 0; i <len; i++){
		if(board[0][i] == 'O'){
			bfsBoundary(board, 0, i, in);
		}
		if(board[wid-1][i] == 'O'){
			bfsBoundary(board, wid-1, i, in);
		}
	}
	for(int i = 0; i < wid; i++){
		if(board[i][0] == 'O'){
			bfsBoundary(board, i, 0, in);
		}
		if(board[i][len-1] == 'O'){
			bfsBoundary(board, i, len-1, in);
		}
	}
	//顺序访问
	for(int i = 0; i < wid; i++){
		for(int j = 0; j < len; j++){
			if(board[i][j] == 'b'){
				board[i][j] = 'O';
			}
			else if(board[i][j] == 'O'){
				board[i][j] = 'X';
			}
		}
	}


}

```
https://leetcode.com/problems/number-of-islands/#/description
