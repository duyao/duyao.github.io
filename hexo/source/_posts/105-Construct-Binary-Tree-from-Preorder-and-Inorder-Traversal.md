title: 105. Construct Binary Tree from Preorder and Inorder Traversal
toc: true
date: 2017-03-20 17:41:38
tags: leetcode
categories: algorithm
---

根据先序中序得到树，这类问题都是用递归来解决的


https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/#/description
```java
public TreeNode buildTree(int[] preorder, int[] inorder) {

    if(preorder == null || preorder.length == 0 || inorder == null || inorder.length == 0){
        return  null;
    }
    return f(preorder, inorder, 0, 0, inorder.length);
}

public TreeNode f(int[] preorder, int[] inorder, int pre, int inl, int inr) {

    //这里所有的区间都是左闭右开
    if (pre >= preorder.length || inl >= inr) {
        return null;
    }
    int r = -1;
    //寻找根在中序的位置
    for (int i = inl; i < inr; i++) {
        if (inorder[i] == preorder[pre]) {
            r = i;
            break;
        }
    }
    TreeNode root = new TreeNode(preorder[pre]);
    //左子树的根节点就是pre的后面一个
    root.left = f(preorder, inorder, pre + 1, inl, r);
    //右子树的根节点就是pre+左子树的长度的后面一个
    root.right = f(preorder, inorder, pre + (r - inl) + 1, r + 1, inr);
    return root;
}
```

https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/#/description
```java
public TreeNode buildTree(int[] inorder, int[] postorder) {

    if (postorder == null || postorder.length == 0 || inorder == null || inorder.length == 0) {
        return null;
    }
    //初始，根的位置在最后
    return f(postorder, inorder, postorder.length-1, 0, inorder.length);
}

public TreeNode f(int[] postorder, int[] inorder, int post, int inl, int inr) {

    //这里所有的区间都是左闭右开
    if (post >= postorder.length || inl >= inr) {
        return null;
    }
    int r = -1;
    //寻找根在中序的位置
    for (int i = inl; i < inr; i++) {
        if (inorder[i] == postorder[post]) {
            r = i;
            break;
        }
    }
    TreeNode root = new TreeNode(postorder[post]);
    //左子树的根节点就是post减去右子树长度的前一个
    root.left = f(postorder, inorder, post - (inr - r - 1) - 1, inl, r);
    //右子树的根节点就是pre的前一个
    root.right = f(postorder, inorder, post - 1, r + 1, inr);
    return root;
}
```
