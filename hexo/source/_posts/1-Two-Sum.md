title: 1. Two Sum
toc: true
date: 2016-03-27 17:16:36
tags:
- leetcode
- algorithm
categories:
---

[1. Two Sum](https://leetcode.com/problems/two-sum/)


```
public int[] twoSum(int[] nums, int target) {
	HashMap<Integer, Integer> map = new HashMap<Integer, Integer>();
	
	for (int i = 0; i < nums.length; i++) {
		if(map.containsKey(target - nums[i])){
			return new int[]{target - nums[i],i};
		}else{
			map.put(nums[i], i);
		}
	}
	return null;

}
```

注意不是有序表，用两个指针不管用，本题目如果硬是凑出了有序表，时间复杂度变为O(nlog(n))