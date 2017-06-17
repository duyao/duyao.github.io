title: 42. Trapping Rain Water
toc: true
date: 2016-05-06 11:49:15
tags: leetcode
categories: algorithm
---

- easy version
[11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)
- hard version
[42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

Both are **Two Pointers**

## 11. Container With Most Water

```
public int maxArea(int[] height) {
	int i = 0;
	int j = height.length-1;
	int res = 0;
	while(i < j){
		//面积
		int tmp = (j-i) * Math.min(height[i], height[j]);
		res = Math.max(res, tmp);
		if(height[i] > height[j]){
			j--;
		}else{
			i++;
		}
	}
	return res;

}
```

## 42. Trapping Rain Water

```
public int trap(int[] height) {
int i = 0;
int j = height.length - 1;
//记录当前最大高度
int curh = 0;
//记录每个值的高度
int[] res = new int[height.length];
while (i <= j) {
	if (height[i] > height[j]) {
		// curh = Math.max(curh, height[i]);
		curh = Math.max(curh, height[j]);
		res[j] = curh;
		j--;
	} else if (height[i] < height[j]) {
		curh = Math.max(curh, height[i]);
		res[i] = curh;
		i++;
	} else {
		curh = Math.max(curh, height[i]);
		res[i] = curh;
		res[j] = curh;
		i++;
		j--;
	}

}
int cnt = 0;
for (int k = 0; k < height.length; k++) {
	cnt += (res[k] - height[k]);
}
return cnt;

}
```

## More Reference

[11. Container With Most Water](https://leetcode.com/discuss/questions/oj/trapping-rain-water?sort=votes)
[42. Trapping Rain Water](https://leetcode.com/discuss/questions/oj/container-with-most-water?sort=votes)
