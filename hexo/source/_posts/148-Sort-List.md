title: 148. Sort List
toc: true
date: 2016-07-16 21:45:40
tags:
- leetcode
- algorithm
categories:
---
# 链表的归并操作
[148. Sort List](https://leetcode.com/problems/sort-list/)
[23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)



## 23. Merge k Sorted Lists
```
public ListNode mergeKLists(ListNode[] lists) {
	return sort(lists,0 ,lists.length-1);
}

private ListNode sort(ListNode[] lists, int s ,int e){
	if(s < e){
		int mid = (s + e) / 2;
		ListNode head1 = sort(lists, s, mid);
		ListNode head2 = sort(lists, mid+1, e);
		return merge(head1, head2);
		
	}else if(s == e){
		return lists[s];
	}else{
		return null;
	}
}

private ListNode merge(ListNode a, ListNode b){
	ListNode head = new ListNode(0);
	ListNode cur = head;
	while(a != null && b != null){
		if(a.val < b.val){
			cur.next = a;
			a = a.next;
		}else{
			cur.next = b;
			b = b.next;
		}
		cur = cur.next;
	}
	while(a != null){
		cur.next = a;
		a = a.next;
		cur = cur.next;
	}
	while(b != null){
		cur.next = b;
		b = b.next;
		cur = cur.next;
	}
	
	return head.next;
	
}
```