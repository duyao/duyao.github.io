title: 101. Symmetric Tree
date: 2016-02-04 11:15:44
tags:
- leetcode
- algorithm
---
[101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)


### 递归式recursively 

```
public static boolean isSymmetric(TreeNode left, TreeNode right) {
	if(left == null && right == null){
		//不需要判断子树，一定对称
		return true;
	}else if(left != null && right != null ){
		if(left.val != right.val){
			//一定不对称，不要需要判断子树
			return false;
		}
	}else{
		return false;
	}
	//左子树的右和右子树的左对比，这里左右子树都存在
	return isSymmetric(right.right, left.left) & isSymmetric(right.left, left.right);
	
}

public static boolean isSymmetric(TreeNode root) {
	if(root == null){
		return true;
	}else{
		//判断左右子树是否对称
		return isSymmetric(root.left, root.right);
	}
	
}
```

### 循环式iteratively

```
public static boolean isSymmetric(TreeNode left, TreeNode right) {
	//左右队列分别进行判断
	LinkedList<TreeNode> rlist = new LinkedList<TreeNode>();
	LinkedList<TreeNode> llist = new LinkedList<TreeNode>();
	rlist.offer(right);
	llist.offer(left);
	TreeNode lNode, rNode;
	while (!rlist.isEmpty() && !llist.isEmpty()) {
		lNode = llist.poll();
		rNode = rlist.poll();
		if(lNode == null && rNode == null){
			continue;
		}else if(lNode != null && rNode != null){
			if(lNode.val != rNode.val){
				return false;
			}else{
				//注意进入队列的顺序和位置
				llist.offer(lNode.left);
				rlist.offer(rNode.right);
				
				llist.offer(lNode.right);
				rlist.offer(rNode.left);
			}
		}else{
			return false;
		}
	}
	
	return true;

}

public static boolean isSymmetric(TreeNode root) {
	//使用两个队列进行遍历
	if(root == null){
		return true;
	}else{
		return isSymmetric(root.left,root.right);
	}

	
}

```


### 层次遍历树，包括空节点（处理为最大值的节点），判断字符串是否对称
> wrong answer

```
// 层次遍历
public static boolean travel(TreeNode root,int tall) {
	String s = "";
	int height = 0;

	if (root != null) {
		LinkedList<TreeNode> list = new LinkedList<TreeNode>();
		list.add(root);
		TreeNode cur = new TreeNode(0);
		TreeNode last = root;
		while (!list.isEmpty() && height != tall) {
			cur = list.poll();
			
			if (cur.val == Integer.MAX_VALUE) {
				
					s += "*";
				
			} else {
				s += cur.val;
				//空节点会是层数出错，因此要变化
				//空节点设置为值最大的节点
				if(cur.left == null){
					list.offer(new TreeNode(Integer.MAX_VALUE));
				}else{
					list.offer(cur.left);

				}
				if(cur.right == null){
					list.offer(new TreeNode(Integer.MAX_VALUE));
				}else{
					list.offer(cur.right);

				}
			}
			
			if(cur == last){
				last = list.getLast();
				height++;
				System.out.println(s+f(s));
				if(!f(s)){
					return false;
				}
				s = "";
			}
			
			
			
		}

	}
//		System.out.println("height = "+height);
	return true;
}

public static boolean f(String s) {
	int i = 0;
	int j = s.length() - 1;
	// 0,1 2,3 4 5 6,

	while (i < j) {
		if (s.charAt(i) != s.charAt(j)) {
			return false;
		}
		i++;
		j--;
	}
	return true;
}

public static boolean isSymmetric(TreeNode root) {
	if(root == null){
		return true;
	}
	
	return travel(root,getTall(root));
	
}

public static int getTall(TreeNode root){
	if(root == null){
		return 0;
	}
	return Math.max(getTall(root.left), getTall(root.right)) + 1;
}
```