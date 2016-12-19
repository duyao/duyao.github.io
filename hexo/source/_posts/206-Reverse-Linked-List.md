title: 206. Reverse Linked List
date: 2016-02-02 10:45:30
tags:
- leetcode
- algorithm

---

[206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

---

这里主要讨论递归方法

```
public static ListNode reverseList(ListNode head) {
	if(head.next == null || head == null){
		return head;
	}else{
		ListNode node = reverseList(head.next);
		//这里反转使用head，而不是node，
		//node 只是用来返回
		//置反
		head.next.next = head;
		//去掉原来的指针
		head.next = null;
		return node;
	}
}

```

