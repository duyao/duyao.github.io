title: 89. Gray Code
toc: true
date: 2016-04-10 17:03:18
tags:
- leetcode
- algorithm
categories:
---
[89. Gray Code](https://leetcode.com/problems/gray-code/)

## 方法一

### 思路

- n = 2

00 -> 01 -> 11 -> 10 

- n = 3

000 -> 001 -> 011 -> 010 ->||||| 110 -> 111 -> 101 -> 100

因此可以看出来求n的格雷码与n-1有关
n的格雷码可以分为两部分
第1部分是第1位，新添加的0或者1
第2部分是后面的数字，可以看出来是n-1的格雷码的结果

而n的结果集是镜面对称的
前一部分是0开头加上n-1结果集
后一部分是1开头加上n-1结果集，
而两部分结果集的顺序相反

### 代码

```
public List<Integer> grayCode(int n) {
	List<String> list = new ArrayList<String>();
	List<Integer> res = new ArrayList<Integer>();
	list.add("");
	if(n == 0){
	    res.add(0);
	    return res;
	}
	for (int i = 0; i < n; i++) {
		List<String> tlist = new ArrayList<String>();
		for (String s : list) {
			String a = "0" + s;
			tlist.add(a);
		}
		//mirror situation
		for (int j = list.size()-1; j >= 0; j--) {
			String s = list.get(j);
			String a = "1" + s;
			tlist.add(a);
		}
		list = tlist;
	}
	for (int i = 0; i < list.size(); i++) {
		int tmp = Integer.parseInt(list.get(i),2);
		res.add(tmp);
	}
	return res;
}
```

Reference : [89. Gray Code](https://leetcode.com/discuss/1525/what-if-i-have-no-knowledge-over-gray-code-before)

## 方法二

```
public List<Integer> grayCode(int n) {
	List<Integer> list = new ArrayList<Integer>();
	for (int i = 0; i < 1 << n; i++) {
		System.out.println(i +" ^ "+(i >> 1) +" = "+(i ^ (i >> 1)));
		list.add(i ^ i >> 1);
	}
	return list;
}
```
Reference : [wikipedia](https://en.wikipedia.org/wiki/Gray_code)