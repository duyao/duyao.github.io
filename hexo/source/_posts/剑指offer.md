title: 剑指offer
toc: true
date: 2017-04-12 21:26:22
tags:
categories:
- algorithm

---
# 二进制中1的个数
http://www.lintcode.com/zh-cn/problem/count-1-in-binary/
原理是n&(n-1)就能把最右边的1变成0
比如：n=5的二进制是101,n-1=4二进制是100，n&(n-1)=101&100=100，这样5中最右边的1就变成了0
```java
public int countOnes(int n) {
    int cnt = 0;
    while (n != 0) {
        cnt++;
        n = n & (n - 1);
    }
    return cnt;
}
```
还可以判断一个是是不是2的指数(2的指数中只有1位是1)
还可以判断一个二进制数变为另一个二进制数需要转换几位(先异或，再求)

# 在O(1)时间复杂度删除链表节点
http://www.lintcode.com/zh-cn/problem/delete-node-in-the-middle-of-singly-linked-list/
![删除链表节点](http://7xilc8.com1.z0.glb.clouddn.com/deletenode.png)
a图是一个链表
b图是已知先序删除链表的过程
c图是只知道要删除节点，那么需要将该删除节点和后继换位置，然后删除后继就可以完成删除节点的过程
```java
public void deleteNode(ListNode node) {
    ListNode next = node.next;
    if(next != null){
        int a = next.val;
        next.val = node.val;
        node.val = a;
    }else{
        node = null;
        return;
    }
    node.next = next.next;

}
```
# 快速幂

# 搜索二维矩阵
http://www.lintcode.com/zh-cn/problem/search-a-2d-matrix-ii/
在每行每列的矩阵中寻找元素个数
这道题目应该在矩阵右上角开始查找，然后划线删除法
下图是查找数字7的过程
![搜索二维矩阵](http://7xilc8.com1.z0.glb.clouddn.com/findinsort.png)
```java
public int searchMatrix(int[][] matrix, int target) {
    if(matrix == null || matrix.length == 0 || matrix[0].length==0){
        return 0;
    }
    // write your code here
    int i = 0;
    int j = matrix[0].length-1;
    int cnt = 0;
    while(i < matrix.length && j>=0){
        if(matrix[i][j] > target){
            j--;
        }else if(matrix[i][j] < target){
            i++;
        }else{
            cnt++;
            j--;
            i++;
        }

    }
    return cnt;
}
```
# 是否是子树
http://www.lintcode.com/zh-cn/problem/subtree/
首先从t1中的每一个值对比t2的根节点，
如果值不相同，就要继续比较t1中的值和t2
如果值相同，就可以比较是不是完全相同的子树了
==如果完全相同的子树返回
==如果不完全相同就继续比较的！！！，即t1.left和t1.right于t2比较
```java
public static boolean isSubtree(TreeNode T1, TreeNode T2) {
      if (T2 == null || T1 == null && T2 == null) {
          return true;
      } else if (T1 == null) {
          return false;
      } else {
          if (T1.val != T2.val) {
              return isSubtree(T1.left, T2) || isSubtree(T1.right, T2);
          } else {
            //这里不能直接返回isSame(T1, T2)，不相同要继续比较
              if (isSame(T1, T2)) {
                  return true;
              } else {
                  return isSubtree(T1.left, T2) || isSubtree(T1.right, T2);
              }
          }
      }


  }

  private static boolean isSame(TreeNode T1, TreeNode T2) {
      if (T1 == null && T2 != null || T2 == null && T1 != null) {
          return false;
      } else if (T1 == null && T2 == null) {
          return true;
      } else if (T1.val != T2.val) {
          return false;
      } else {
          return isSame(T1.left, T2.left) && isSame(T1.right, T2.right);
      }
  }

```
