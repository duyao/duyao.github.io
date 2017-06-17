title: 138. Copy List with Random Pointer
toc: true
date: 2017-03-06 20:08:18
tags: leetcode
categories: algorithm
---
[138. Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer)

https://leetcode.com/problems/clone-graph/?tab=Description
可以和复制图一样，弄个map放节点，一次复制next,一次复制random
与复制图相比，复制链表是可以做到o(1)空间的

做法是遍历3次，
1）复制原节点，使得新节点是原节点的next
2）添加新节点的random
3）在原节点中删去添加的节点
前两次新节点都不是直接相连的，而是通过原节点相连接的
方法如图所示
![方法](http://7xilc8.com1.z0.glb.clouddn.com/Q138.png)

代码有问题


类似题目
https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree
```
public TreeNode sortedListToBST(ListNode head) {
    if (head == null) {
        return null;
    }
   return addNode(head, null);


}

public TreeNode addNode(ListNode head, ListNode tail) {
    //边界条件是这个队列中只有一个元素
    if(head == tail){
        return  null;
    }

    ListNode slow = head;
    ListNode fast = head;
    while (slow.next != tail && fast != tail && fast.next != tail) {
        slow = slow.next;
        fast = fast.next.next;
    }
    TreeNode root = new TreeNode(slow.val);
    root.left = addNode(head, slow);
    root.right = addNode(slow.next, tail);
    return root;


}
```
