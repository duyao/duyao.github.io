title: 9. Palindrome Number
toc: true
date: 2016-04-13 22:28:40
tags:
- leetcode
- algorithm
categories:
---

[9. Palindrome Number](https://leetcode.com/problems/palindrome-number/)



## key points 

- negative numbers
- overflow
- no extra space

The break point is `sum >= x` --> `(sum ==x)||(sum/10==x)`

## source code

```
public boolean isPalindrome(int x) {
    if(x < 0 || (x!=0 && x%10==0)){
        return false;
    }
    int sum = 0;
    while(x > sum){
        sum = x%10 + sum*10;
        x/=10;
    }
    return (sum ==x)||(sum/10==x);
}
```