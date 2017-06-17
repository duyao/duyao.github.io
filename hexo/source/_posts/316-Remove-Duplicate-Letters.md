title: 316. Remove Duplicate Letters
toc: true
date: 2016-05-12 17:06:52
tags:
- leetcode
- algorithm
categories:
---
[316. Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/)

## 区间寻找
### 思路
找到每个字母出现的最后位置，把整个串化为若干个区间，
找到每个区间的最小值，然后把字符串中最小值删除

**TIPS**

- 区间开始一定是最小值第一次出现的地方
- 区间结束是该原来的end或者新的end

### 代码
```
public static int findMinPos(HashMap<Character, Integer> map) {
	int pos = Integer.MAX_VALUE;
	for (char c : map.keySet()) {
		int x = map.get(c);
		if (x < pos) {
			pos = x;
		}
	}
	return pos;
}

/**
 * 找到所有字母最后出现的位置，然后按照这些位置依次划分区间，找到每个区间中最小的字母
 * 
 * @param s
 * @return
 */
public String removeDuplicateLetters(String s) {
	if (s == null || s.length() == 0) {
		return "";
	}
	HashMap<Character, Integer> map = new HashMap<Character, Integer>();
	char[] c = s.toCharArray();
	for (int i = 0; i < c.length; i++) {
		map.put(c[i], i);
	}
	int start = 0, end = findMinPos(map);
	char[] res = new char[map.size()];
	char min = 'z' + 1;
	for (int j = 0; j < res.length; j++) {
		// 区间内部找最小值
		for (int i = start; i <= end; i++) {
			//这里一定是c[i] < min，无等号，这样才能找到一个min出现的位置
			if (c[i] < min && map.containsKey(c[i])) {
				min = c[i];
				start = i + 1;
			}
		}
		// 先remove掉min，后面才可以重新查找
		res[j] = min;
		map.remove(min);
		
		if (min == c[end]) {
			end = findMinPos(map);
		}

		// 重置min
		min = 'z' + 1;
	}

	return new String(res);

}
```

### reference
[easy-to-understand-iterative-java-solution](https://leetcode.com/discuss/73777/easy-to-understand-iterative-java-solution)

## 栈
### 思路
- int[] count: 记录每个字母出现的次数
- boolean[] b: 是否已经被记录过
- Stack<Character> stack: 结果集
当栈中栈顶元素比当前元素大，且元素没有被记录过，以后还会出现的，出栈顶
即栈中元素就是结果集，比栈顶大的元素才可以进栈

### 代码
```
public String removeDuplicateLetters(String s) {
	int[] count = new int[26];
	boolean[] b = new boolean[26];
	Stack<Character> stack = new Stack<Character>();
	char[] c = s.toCharArray();
	for (int i = 0; i < c.length; i++) {
		count[c[i]-'a']++;
	}
	for (int i = 0; i < c.length; i++) {
		count[c[i]-'a']--;
		while(!stack.isEmpty() && stack.peek() > c[i] && !b[c[i]-'a']&& count[stack.peek()-'a'] > 0){
			b[stack.peek()-'a'] = false;
			stack.pop();
			
		}
		if(!b[c[i]-'a']){
			stack.push(c[i]);
			b[c[i]-'a'] = true;
		}
	}
	StringBuffer sb = new StringBuffer();
	while(!stack.isEmpty()){
		sb.insert(0,stack.pop());
	}
	return sb.toString();
}
```

### reference
[java-solution-using-stack-with-comments](https://leetcode.com/discuss/75738/java-solution-using-stack-with-comments)