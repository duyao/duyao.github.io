title: 230. Kth Smallest Element in a BST
toc: true
date: 2016-03-26 22:58:58
tags: 
- leetcode
- algorithm
categories:
---

[230. Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

```
public int kthSmallest(TreeNode root, int k) {
	int cnt = vis(root.left);
	if(k == cnt+1){
		return root.val;
	}else if(k < cnt+1){
		从左子树找
		return kthSmallest(root.left, k);
	}else{
		//从右子树找
		return kthSmallest(root.right, k - cnt - 1);
	}
	
}

//计算节点个数
public int vis(TreeNode root) {
	if (root == null) {
		return 0;
	}
	return vis(root.left) + vis(root.right) + 1;
}
```