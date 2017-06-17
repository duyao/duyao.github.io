title: 448. Find All Numbers Disappeared in an Array
toc: true
date: 2017-03-13 19:46:43
tags: leetcode
categories: algorithm
---
[448. Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array)
[442. Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array)

两道非常相似的题目
都是使用`nums[nums[i]-1] = -nums[nums[i]-1]`，前提是这个数组是[1,n]
原理是数组中的下标是[0,n-1]，然后通过上述的转换，即访问过得数字变为了负数，其中减一的原因是数组和下标相差1

448找重复
```java
public List<Integer> findDuplicates(int[] nums) {

    List<Integer> res = new ArrayList<>();
    for(int i = 0; i < nums.length; i++){
        int val = Math.abs(nums[i])-1;
        if(nums[val] > 0){
            nums[val] = -nums[val];
        }else{
            res.add(val+1);
        }
    }
    return res;


}
```
442找没有出现的
```java
public List<Integer> findDisappearedNumbers(int[] nums) {
    List<Integer> ret = new ArrayList<Integer>();

    for(int i = 0; i < nums.length; i++) {
        int val = Math.abs(nums[i]) - 1;
        if(nums[val] > 0) {
            nums[val] = -nums[val];
        }
    }

    for(int i = 0; i < nums.length; i++) {
        if(nums[i] > 0) {
            ret.add(i+1);
        }
    }
    return ret;
}
```

http://blog.csdn.net/yutianzuijin/article/details/11384507
