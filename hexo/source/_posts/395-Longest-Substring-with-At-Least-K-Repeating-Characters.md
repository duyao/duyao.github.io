title: 395. Longest Substring with At Least K Repeating Characters
toc: true
date: 2017-03-22 20:45:03
tags: leetcode
categories: algorithm
---
https://leetcode.com/problems/longest-substring-with-at-least-k-repeating-characters/#/description

类似于二分的做法
找出小于k次出现的字符，然后以该字符二分区间，从而找到最长
```java

public int longestSubstring(String s, int k) {
    if (s == null || s.length() == 0 || k < 1) {
        return s.length();
    }

    return f(s, 0, s.length(), k);
}

//找出每一段区间中那个少于k次的值，然后二分法求得每一段的长度
public int f(String s, int a, int b, int k) {
    if (b < a) return 0;
    if (b - a + 1 < k) return 0;

    //记录下来每一个字符出现的频率
    int[] arr = new int[26];
    for (int i = a; i < b; i++) {
        arr[s.charAt(i) - 'a']++;
    }

    int len = b - a;
    for (int i = a; i < b; i++) {

        //找到小于k次的字符，然后用它分割整个区间
        if (arr[s.charAt(i) - 'a'] < k && arr[s.charAt(i) - 'a'] > 0) {
            //二分法求值
            int left = f(s, i + 1, b, k);
            int right = f(s, a, i, k);

            return Math.max(right, left);
        }
    }

    return len;

}
```
