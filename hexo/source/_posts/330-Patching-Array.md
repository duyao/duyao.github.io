title: 330. Patching Array
date: 2016-03-11 18:11:17
tags:
- leetcode
- algorithm
toc: true
---

[330. Patching Array](https://leetcode.com/problems/patching-array/)

## 思路
贪心
设置miss为缺少的数字，sum为**当前所有选中数字的的最小和**
挑选miss的规则

- 若数组中x <= sum + 1则选中x
- 若数组没有，就选中sum + 1为miss
直到sum > n

[思路参考](https://leetcode.com/discuss/82822/solution-explanation)

```
public int minPatches(int[] nums, int n) {

	long miss = 0;
	long sum = 0;
	int cnt = 0;
	int i = 0;

	while (sum < n) {
		if (i < nums.length && nums[i] <= sum + 1) {
			sum += nums[i++];
		} else {
			miss = sum + 1;
			sum += miss;
			cnt++;
		}
	}
	return cnt;
}
```











