title: 3. Longest Substring Without Repeating Characters
toc: true
date: 2017-03-20 22:31:55
tags: leetcode
categories: algorithm
---
https://leetcode.com/problems/longest-substring-without-repeating-characters/#/solutions

这道题目就是用map存放已经遍历过字母的位置
如果发现存在就重新设置起点的位置，注意设置起点位置是在当前起点和出现字符位置+1中选择大的，防止回退
另外每次都要更新maxl，而不是只更换起点时候更新，因为一直没有重复的就不会发生更新
最后计算长度的时候还要+1，因为都是闭区间

```java
public int lengthOfLongestSubstring(String s) {
   HashMap<Character, Integer> map = new HashMap<>();
   int start = 0, cur = 0;
   int maxl = 0;
   while (cur < s.length()) {
       if (map.containsKey(s.charAt(cur))) {
           //注意这里不能单纯找后一个值，而是要比较当前值与回退值哪个大
           //比如abba
           start = Math.max(start, map.get(s.charAt(cur)) + 1);
       }

       //这里每次都要更新位置
       map.put(s.charAt(cur), cur);
       //全部都是闭区间
       maxl = Math.max(maxl, cur - start + 1);
       cur++;

   }
   // System.out.println(maxl);
   return maxl;


}
```
