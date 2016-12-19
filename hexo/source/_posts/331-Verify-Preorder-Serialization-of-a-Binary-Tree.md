title: 331. Verify Preorder Serialization of a Binary Tree
date: 2016-03-09 21:08:31
tags:
- leetcode
- algorithm
toc: true
---
[331. Verify Preorder Serialization of a Binary Tree](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/)

## 思路一
砍掉叶子节点
两个叶子节点访问后就将该父节点置为叶子节点，这样一个树访问完成后只剩下一个叶子节点
因此需要一个栈来存放先序遍历，对于连续栈中两个##，删除##，并且将栈顶替换为#
```
public boolean isValidSerialization(String preorder) {
	if(preorder == null || preorder.length() == 0){
		return false;
	}
	Stack<String> stack = new Stack<String>();
	String[] s = preorder.split(",");
	for (String string : s) {
		//注意这里是while，因为遇到连续##就要替换
		while(string.equals("#") && !stack.isEmpty() && stack.peek().equals("#")){
			//将#pop
			stack.pop();
			if(stack.empty()){
				return false;
			}
			//将栈顶元素替换为#
			stack.pop();
		}
		stack.push(string);
		
	}
	if(stack.size() == 1 && stack.peek().equals("#")){
		return true;
	}
	return false;
}
```
对于后序遍历，是遇到数字替换连续##

## 思路二
入度与出度的差
对于一个树来说分为叶子节点和非叶节点

- 非叶子结点:出度2，入度1(不包括根结点)
- 叶子结点：出度0，入度1

因此diff = 出度 - 入度 = 1，因为根结点入度为0
所以对于一个正确的树，一定会有diff = 1
且对于任何一个节点，diff都是非负数
对于每一个节点进入，diff--，而若是非叶子节点，diff+=2

```
public boolean isValidSerialization(String preorder) {
	String[] nodes = preorder.split(",");
	int diff = 1;
	for (String node : nodes) {
		if (--diff < 0)
			return false;
		if (!node.equals("#"))
			diff += 2;
	}
	return diff == 0;

}
```

