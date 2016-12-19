title: 315. Count of Smaller Numbers After Self
toc: true
date: 2016-05-10 21:20:45
tags:
- leetcode
- algorithm
categories:
---

[315. Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/)

多种解法

## 二分搜索

### 思路

- list存储有序的数字

从最后开始，将该数组分为两部分
一部分是没有排序的前面
一部分是有序的后面，存储在list中

对于位置i的元素x
利用二分找到x应该插入的list的坐标po，因为list 有序，所以po同时也表示在list中有po个数字小于x
则这个po就是应该整个数组中小于x的右边元素的个数

### 代码
```
//binary search
public static List<Integer> countSmaller(int[] nums) {
	List<Integer> list = new ArrayList<Integer>();
	if(nums == null || nums.length == 0){
		return list;
	}
	//init
	List<Integer> sorted = new ArrayList<Integer>();
	sorted.add(nums[nums.length-1]);
	list.add(0);

	for(int i = nums.length-2; i >= 0; i--){
		int po = getIndex(nums[i], sorted);
		sorted.add(po,nums[i]);
		list.add(po);
	}
	//reverse
	Collections.reverse(list);
	return list;
	
}
//smaller exclude equal
public static int getIndex(int val, List<Integer> sorted){
	int i = 0;
	int j = sorted.size();
	int mid;
	while(i < j){
		mid = (i+j)/2;
		if(val <= sorted.get(mid)){
			j = mid;
		}else{
			i = mid+1;
		}
		
	}
	return i;
}
```

## Merge Sort

### 思路
利用merge sort思想，排序保存下标在index中
将数组分为左右两个组，左边组合并时候要加上右边组合并的个数

- 过程分析
int[] nums = {12,34,4,3,7,5,2,6,1};
=====
start = 0, end = 1
nums->12,34,4,3,7,5,2,6,1,
index->0,1,2,3,4,5,6,7,8,
result->0,0,0,0,0,0,0,0,0,
=====
start = 0, end = 2
nums->12,34,4,3,7,5,2,6,1,
index->2,0,1,3,4,5,6,7,8,
result->1,1,0,0,0,0,0,0,0,
=====
start = 3, end = 4
nums->12,34,4,3,7,5,2,6,1,
index->2,0,1,3,4,5,6,7,8,
result->1,1,0,0,0,0,0,0,0,
=====
start = 0, end = 4
nums->12,34,4,3,7,5,2,6,1,
index->3,2,4,0,1,5,6,7,8,
result->3,3,1,0,0,0,0,0,0,
=====
start = 5, end = 6
nums->12,34,4,3,7,5,2,6,1,
index->3,2,4,0,1,6,5,7,8,
result->3,3,1,0,0,1,0,0,0,
=====
start = 7, end = 8
nums->12,34,4,3,7,5,2,6,1,
index->3,2,4,0,1,6,5,8,7,
result->3,3,1,0,0,1,0,1,0,
=====
start = 5, end = 8
nums->12,34,4,3,7,5,2,6,1,
index->3,2,4,0,1,8,6,5,7,
result->3,3,1,0,0,2,1,1,0,
=====
start = 0, end = 8
nums->12,34,4,3,7,5,2,6,1,
index->8,6,3,2,5,7,4,0,1,
result->7,7,3,2,4,2,1,1,0,
=====
[7, 7, 3, 2, 4, 2, 1, 1, 0]

### 代码
```
public List<Integer> countSmaller(int[] nums) {
	if (nums == null || nums.length == 0) {
		return new ArrayList<Integer>();
	}
	int[] result = new int[nums.length];
	int[] sorted = new int[nums.length];
	// 对该数组排序，存储的是该数字的序号，而不是值
	for (int i = 0; i < nums.length; i++) {
		sorted[i] = i;
	}
	mergeSort(0, nums.length - 1, nums, sorted, result);
	List<Integer> list = new ArrayList<Integer>();
	for (int i = 0; i < result.length; i++) {
		list.add(result[i]);
	}

	return list;
}

public void mergeSort(int start, int end, int[] nums, int[] sorted,
		int[] result) {
	if (end <= start) {
		return;
	}
	int mid = (start + end) / 2;
	mergeSort(start, mid, nums, sorted, result);
	mergeSort(mid + 1, end, nums, sorted, result);

	merge(start, end, nums, sorted, result);

}

public void merge(int start, int end, int[] nums, int[] sorted, int[] result) {
	int mid = (start + end) / 2;
	int leftIndex = start;
	int rightIndex = mid + 1;
	
	// 记录右边合并过的个数
	int rightCnt = 0;
	
	//记录本次排序好的下标
	int[] newIndex = new int[end - start + 1];
	int cnt = 0;
	
	while (leftIndex <= mid && rightIndex <= end) {
		//比较的时候时根据已经排好序的数字进行比较
		if (nums[sorted[leftIndex]] <= nums[sorted[rightIndex]]) {
			//记录较小值的下标
			newIndex[cnt] = sorted[leftIndex];
			//result中仍然是对应下标的小于right值
			//分为两组，左边组合并时，加上右边已经合并的个数
			result[sorted[leftIndex]] += rightCnt;
			leftIndex++;
		} else {
			newIndex[cnt] = sorted[rightIndex];
			rightIndex++;
			rightCnt++;
		}
		cnt++;
	}
	//剩余合并
	while(leftIndex <= mid){
		newIndex[cnt++] = sorted[leftIndex];
		result[sorted[leftIndex]] += rightCnt;
		leftIndex++;
	}
	while(rightIndex <= end){
		newIndex[cnt++] = sorted[rightIndex];
		rightIndex++;
	}
	//更新排序
	for(int i = start; i <= end; i++){
        sorted[i] = newIndex[i - start];
    }
}
```


### reference
[11ms-java-solution-using-merge-sort-with-explanation](https://leetcode.com/discuss/74110/11ms-java-solution-using-merge-sort-with-explanation)
[nlogn-time-space-mergesort-solution-with-detail-explanation](https://leetcode.com/discuss/73509/nlogn-time-space-mergesort-solution-with-detail-explanation)


## BST


http://www.cnblogs.com/yrbbest/p/5068550.html




