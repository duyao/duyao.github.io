title: 235.Lowest Common Ancestor of a Binary Search Tree
date: 2016-02-01 10:20:42
tags: 
- leetcode
- algorithm

---

[235.Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

---
## 思路一

记录父节点，,遍历得到每个节点的所有结点，依次比较

```
//遍历过程生成父节点
public static void travle(TreeNode root, HashMap<TreeNode, TreeNode> map){
	if(root != null){
		if(root.left != null){
			map.put(root.left, root);
		}
		if(root.right != null){
			map.put(root.right, root);
		}
		
		travle(root.left, map);
		travle(root.right, map);
	}
}

//得到所有的父节点
public static ArrayList<TreeNode> getParents(HashMap<TreeNode, TreeNode> map, TreeNode root ){
	ArrayList<TreeNode> list = new ArrayList<TreeNode>();
	while(root != null){
		root = map.get(root);
		if(root != null){
			list.add(root );
		}
	}
	return list;
}

public static TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
	//key是当前结点,value是父节点
	HashMap<TreeNode, TreeNode> map = new HashMap<TreeNode, TreeNode>();
	travle(root, map);
	map.put(root, null);
	
	ArrayList<TreeNode> pNode = getParents(map, p);
	ArrayList<TreeNode> qNode = getParents(map, q);
	
	for (TreeNode treeNode : qNode) {
		if(treeNode == p){
			return p;
		}
	}
	for (TreeNode treeNode : pNode) {
		if(treeNode == q){
			return q;
		}
	}
	
	
	for (TreeNode qtreeNode : qNode) {
		for (TreeNode ptreeNode : pNode) {
			if(qtreeNode == ptreeNode){
				return qtreeNode;
			}
		}
	}
	
	return null;	
}
```

## 思路二

上面的方法完全没有用到BST的性质
BST(Binary Search Tree)左小右大
对于查找BST中两个节点的公共父节点，该结果值一定介于两个节点值之间(包括其本身)
因此只要遍历该BST，找到第一个介于两者之间的值就是结果

```
public static void travle(TreeNode root, TreeNode p, TreeNode q,
		ArrayList<TreeNode> list) {

	if (root != null) {
		//只有介于两者之间的值才加入队列
		if (root.val >= p.val && root.val <= q.val) {
			list.add(root);
		}

		//遍历所有的值，不能选择，否则会出错，在加入时选择
		travle(root.left, p, q, list);
		travle(root.right, p, q, list);

	}
}

public static TreeNode lowestCommonAncestor(TreeNode root, TreeNode p,
		TreeNode q) {
	if (q.val == p.val) {
		return q;
	}
	TreeNode small = p.val < q.val ? p : q;
	TreeNode big = p.val < q.val ? q : p;
	ArrayList<TreeNode> list = new ArrayList<TreeNode>();
	travle(root, small, big, list);
	if (list.size() == 0) {
		return big;
	} else {
		return list.get(0);
	}

}

public static void main(String[] args) {
	// [5,3,6,2,4,null,null,1]
	TreeNode root = new TreeNode(5);
	TreeNode t2 = new TreeNode(3);
	TreeNode t3 = new TreeNode(6);
	TreeNode t4 = new TreeNode(2);
	TreeNode t5 = new TreeNode(4);
	TreeNode t6 = new TreeNode(1);
	root.left = t2;
	root.right = t3;
	t2.left = t4;
	t2.right = t5;
	t4.left = t6;
	TreeNode res = lowestCommonAncestor(root, t5, t4);
	System.out.println(res.val);

}
```



