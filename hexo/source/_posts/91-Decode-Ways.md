title: 91. Decode Ways
toc: true
date: 2016-04-08 20:54:24
tags:
- leetcode
- algorithm
categories:
---

[91. Decode Ways](https://leetcode.com/problems/decode-ways/)

## 方法一

暴力递归--超时

```
public int numDecodings(String s) {
	if(s == null || s.length() == 0){
		return 0;
	}else if(s.length() == 1){
		return 1;
	}
	List<List<Integer>> res = new ArrayList<List<Integer>>();
	int sum = f(s, 0, 1, new ArrayList<Integer>()) + f(s, 0, 2, new ArrayList<Integer>());
	System.out.println(sum);
	return sum;
}

public int f(String s, int i, int j, List<Integer> path) {
	int cnt = 0;
	//System.out.println("i=" + i + ",j=" + j);
	if (j == s.length()) {
		Integer tmp = Integer.valueOf(s.substring(i, j));
		if (tmp > 0 && tmp < 27) {
			path.add(tmp);
			cnt++;
			path.remove(path.size() - 1);
		}
		
	} else {

		Integer tmp = Integer.valueOf(s.substring(i, j));
		if (tmp > 0 && tmp < 27) {
			path.add(tmp);
			if (j + 1 <= s.length()) {
				cnt += f(s, j, j + 1, path);
			}
			if (j + 2 <= s.length()) {
				cnt += f(s, j, j + 2, path);
			}
			path.remove(path.size() - 1);
		}

	}
	return cnt;

}
```

### 相似题目
利用递归寻找所有方法[131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)
解法[Q131.java](https://github.com/duyao/MyLeetcode/blob/master/leetcode/src/com/dy/leetcode/Q131.java)


## 方法二

dp从后向前面遍历
递推公式
```
memo[i] = (Integer.parseInt(s.substring(i, i + 2)) <= 26) ? memo[i + 1] + memo[i + 2] : memo[i + 1]; 
```

比如1314

|i|0 |1 |2 |3 |4 |
||-|-|-|-|-|
|a[i]|1| 3| 1| 4|-|
|dp  |4| 2| 2| 1| 1|
|add|`{1},{4}`;`{14}`|`{3}`|`{1},{4}`;`{14}`|-|-|


```
public int numDecodings(String s) {
	int n = s.length();
	if (n == 0)
		return 0;

	int[] memo = new int[n + 1];
	memo[n] = 1;
	memo[n - 1] = s.charAt(n - 1) != '0' ? 1 : 0;

	for (int i = n - 2; i >= 0; i--)
		if (s.charAt(i) == '0')
			continue;
		else {
			memo[i] = (Integer.parseInt(s.substring(i, i + 2)) <= 26) ? memo[i + 1]
					+ memo[i + 2]
					: memo[i + 1];
		}
	return memo[0];
}
```



