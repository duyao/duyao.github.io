title: 451. Sort Characters By Frequency
toc: true
date: 2017-03-20 21:15:19
tags: leetcode
categories: algorithm
---
https://leetcode.com/problems/sort-characters-by-frequency/#/description

其实这是一个位图的思想
首先，数组下标表示字符，而值表示出现的次数
然后新建一个数组，和前一步相反，下标表示的次数，而值表示字符本身
最后遍历第二个数组，因为下标就是次序，这个遍历的过程就已经排序了

还有个小问题就是char和int的转换，java中直接就可以转换了
```
int a = 97
char c = (Char)97;
```

```java
public String frequencySort(String s) {
   if (s == null || s.length() == 0) {
       return s;
   }
   int maxf = 0;
   //首先算出每个值出现的频率
   int fren[] = new int[128];
   for (int i = 0; i < s.length(); i++) {
       int cur = s.charAt(i);
       fren[cur]++;
       maxf = Math.max(maxf, fren[cur]);
   }
   //然后新建一个数组把频率作为下标，值就是字母的list，因为可能多个值出现同样的次数
   //这个数组是一个list数组
   ArrayList<Character>[] list = new ArrayList[maxf + 1];
   for (int i = 0; i < 128; i++) {
       if (fren[i] != 0) {
           if (list[fren[i]] == null) {
               list[fren[i]] = new ArrayList<>();
           }
           list[fren[i]].add((char) i);

       }
   }


   //遍历新建的数组
   StringBuffer sb = new StringBuffer();
   for (int i = maxf; i >= 0; i--) {
       if (list[i] != null) {
           for (int k = 0; k < list[i].size(); k++) {
               for (int j = 0; j < i; j++) {
                   sb.append(list[i].get(k));
               }
           }


       }
   }

   return sb.toString();

}
```


同样的题目还有347
https://leetcode.com/problems/top-k-frequent-elements/#/description
