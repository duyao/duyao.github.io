title: 93. Restore IP Addresses
toc: true
date: 2016-04-01 21:22:22
tags:
- leetcode
- algorithm
categories:
---
[93. Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/)

Backtracking - 回溯

```
public List<String> restoreIpAddresses(String s) {
	List<String> list = new ArrayList<String>();
	if(s == null || s.length() < 4){
		return list;
	}
	for(int i = 1; i < 4;i++){
		String s1 = s.substring(0, i);
		//注意判断是否超出数组长度，且每次循环前判断是否合法，这样能减少后面无用的判断
		for(int j = 1; isVaild(s1) && j < 4 && i+j < s.length();j++){
			String s2 = s.substring(i, i+j);
			for(int k = 1; isVaild(s2) && k < 4 && i+j+k < s.length(); k++){
				String s3 = s.substring(i+j, i+j+k);
				String s4 = s.substring(i+j+k, s.length());
				if(isVaild(s3) && isVaild(s4)){
					list.add(s1+"."+s2+"."+s3+"."+s4);
				}
			}
		}
	}
	return list;
}
public boolean isVaild(String s){
	if(s.length() == 1){
		return true;
	}else if(s.length() == 2 && s.charAt(0) != '0'){
		return true;
	}else if(s.length() == 3 && s.charAt(0) != '0' && Integer.valueOf(s) < 256){
		return true;
	}else{
		return false;
	}
}
```