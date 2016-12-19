title: 169. Majority Element
date: 2016-02-01 11:03:31
tags:
- leetcode
- algorithm
---

[169. Majority Element](https://leetcode.com/problems/majority-element/)

--- 

## 思路1

排序然后找打中间值

## 思路2

```
public int majorityElement(int[] nums) {
	//记录次数cnt和当前数值value
	//数值相同，cnt++,否则cnt--
	//当cnt为0,重置value和cnt
	int value = nums[0];
	int cnt = 1;
	for (int i = 1; i < nums.length; i++) {
		if(value == nums[i]){
			cnt++;
		}else{
			cnt--;
			if(cnt == 0){
				value = nums[i];
				cnt++;
			}
		}
	}
	return value;
}
```


