title: 235.Lowest Common Ancestor of a Binary Search Tree
date: 2016-02-01 10:20:42
tags:
- leetcode
- algorithm

---

[235.Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

---

## 思路二

上面的方法完全没有用到BST的性质
BST(Binary Search Tree)左小右大
对于查找BST中两个节点的公共父节点，该结果值一定介于两个节点值之间(包括其本身)
因此只要遍历该BST，找到第一个介于两者之间的值就是结果

```
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if(root.val > p.val && root.val > q.val){
        return lowestCommonAncestor(root.left, p, q);
    }else if(root.val < p.val && root.val < q.val){
        return lowestCommonAncestor(root.right, p, q);
    }else{
        return root;
    }
}
```


[236. Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree)
```
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
		//树空或者本身就是公共祖先返回的是根节点
    if(root == null || root == p || root == q)  return root;
		//找左右子树的公共祖先
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
		//如果存在，那就一定是
    if(left != null && right != null)   return root;
		//有一边不存在，说明两个节点都在同一边
    return left != null ? left : right;
}
```
