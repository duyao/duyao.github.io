title: 22. Generate Parentheses
toc: true
date: 2016-04-09 22:34:00
tags:
- leetcode
- algorithm
categories:
---
[22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)


## 方法一

### 思路
`addPairs`添加括号
`l`表示该字符串s末尾还可以添加左括号的个数
`r`表示该字符串s末尾还可以添加右括号的个数

对于添加括号的过程

1. 如果r不为0，则r-1，这是添加右括号的过程
2. 如果l不为0，则先l-1，这是添加左括号的过程，然后r+1，因为添加左括号，就要有对应的右括号进行匹配
```
public void addPairs(List<String> res, String s, int l, int r){
	if(l == 0 && r == 0){
		res.add(s);
		return;
	}
	if(r > 0){
		addPairs(res, s+")", l, r-1);
	}
	if(l > 0){
		addPairs(res, s+"(", l-1, r+1);
	}
}
```

调用过程

|string|l|r|
|--|--|--|
||3|0|
|(|2|1|
|()|2|0|
|()(|1|1|
|()()|1|0|
|()()(|0|1|
|()()()|0|0|
|()((|0|2|
|()(()|0|1|
|()(())|0|0|
|((|1|2|
|(()|1|1|
|(())|1|0|
|(())(|0|1|
|(())()|0|0|
|(()(|0|2|
|(()()|0|1|
|(()())|0|0|
|(((|0|3|
|((()|0|2|
|((())|0|1|
|((()))|0|0|

### 代码
```
public List<String> generateParenthesis(int n) {
	List<String> res = new ArrayList<String>();
	addPairs(res, "", n, 0);
	return res;
}
public void addPairs(List<String> res, String s, int l, int r){
	System.out.println("|"+s+"|"+l+"|"+r+"|");
	if(l == 0 && r == 0){
		res.add(s);
		return;
	}
	if(r > 0){
		addPairs(res, s+")", l, r-1);
	}
	if(l > 0){
		addPairs(res, s+"(", l-1, r+1);
	}
}
```

## 方法二

### 思路
全排列的思路

[46. Permutations](http://duyao.github.io/2016/02/18/46-Permutations/)
把一对括号看成一个整体插入，利用`StringBuffer.insert(int offSet, String s)`

### 代码
```
public List<String> generateParenthesis(int n) {
	Set<String> res  = new HashSet<String>();
	res.add("");
	for (int i = 0; i < n; i++) {
		Set<String> tres  = new HashSet<String>();
		for (int j = 0; j <= i; j++) {
			for (String string : res) {
				StringBuffer sb = new StringBuffer(string);
				sb.insert(j,"()");
				tres.add(sb.toString());
			}
		}
		res = tres;
	}
	return new ArrayList<String>(res);
}
```

## 延伸
如果此题目只要求求出方法个数，用dp
思路类似于[96. Unique Binary Search Trees](https://leetcode.com/discuss/24282/dp-solution-in-6-lines-with-explanation-f-i-n-g-i-1-g-n-i)

图示中#代表空位, n表示有括号的对数, f(n)表示有n对括号时的方法总数

- n = 0 //1

- n = 1 //1
() 

- n = 2 //2

()## //2.1 f(0)*f(1)
(##) //2.2 f(1)*f(0)

- n = 3

()#### //3.1 f(0)*f(2)
(##)## //3.2 f(1)*f(1)
(####) //3.3 f(2)*f(0)

- n = 4

()###### //4.1 f(0)*f(3)
(##)#### //4.2 f(1)*f(2)
(####)## //4.3 f(2)*f(1)
(######) //4.4 f(3)*f(0)

比如n = 4,
4.1中，1对括号相隔0个位置，剩余3对括号的排列方法就是f(3)
4.2中，1对括号相隔1个位置，则被分为2个批次，一批是f(1)，另一批是f(2)
...

因此得到递推公式
f(n) = f(0) \* f(n-1) + f(1) \* f(n-2) + f(2) \* f(n-3) + ... + f(n-2) \* f(1) + f(n-1) \* f(0) 
f(0) = 1


