title: 307. Range Sum Query - Mutable
toc: true
date: 2016-04-05 22:00:14
tags:
- leetcode
- algorithm
categories:
---
[307. Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/)

## 位运算

|运算|含义|
|:--:|:--:|
|x>>1|右移一位，x /= 2|
|x<<1|左移一位，x *= 2|
|`x|1`|或，如果x = 2\*i,那么`x|1` = 2*i +1|
|x^1|异或，奇偶互换|
|x&1|是否为奇数，x%2|

## 线段树

[segment tree](http://codeforces.com/blog/entry/18051)

原型是二叉树，叶子节点是数组元素，非叶子节点代表一段序列的和


## 代码

```
public class NumArray {
     int[] array;
	    int n;

    public NumArray(int[] nums) {
        n = nums.length;
        array = new int[2*n];
        //init leaves
        for(int i = 0; i < nums.length; i++){
            array[n+i] = nums[i];
        }
        //init node
        for(int i = n - 1; i >= 0; i--){
            array[i] = array[i << 1] + array[i << 1 | 1];
        }
        
    }

    void update(int i, int val) {
        array[i+n] = val;
        for(int j = i+n; j > 0; j >>= 1){
            array[j >> 1] = array[j] + array[j ^ 1];
        }
        
    }

    public int sumRange(int i, int j) {
        int sum = 0;
        for(int l = i+n, r = j+n; l < r; l >>= 1 , r >>= 1){
            //if left is right child, sum does not contain parent
            if((l&1) == 1){
                sum += array[l++];
            }
            if((r&1) == 1){
                sum += array[--r];
            }
        }
        return sum+array[j+n];
        
    }
}
```



