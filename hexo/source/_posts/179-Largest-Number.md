title: 179. Largest Number
toc: true
date: 2016-05-05 21:43:16
tags:
- leetcode
- algorithm
categories:
---
[179. Largest Number](https://leetcode.com/problems/largest-number/)

```
public class cp implements Comparator<String>{
		@Override
	public int compare(String o1, String o2) {
		//逆序
	    return -1*(o1+o2).compareTo(o2+o1);
	}
		
	
}

public String largestNumber(int[] nums) {
	StringBuffer res = new StringBuffer();
	if(nums == null || nums.length == 0){
		return res.toString();
	}
	ArrayList<String> list = new ArrayList<String>();
	for (int i = 0; i < nums.length; i++) {
		list.add(String.valueOf(nums[i]));
	}
	Collections.sort(list, new cp());
	for (String string : list) {
		res.append(string);
	}
	if(res.toString().startsWith("0")){
		return "0";
	}
	
	return res.toString();
	

}
```