title: 49. Group Anagrams
date: 2016-02-18 09:09:37
tags:
- leetcode
- algorithm
---

[49. Group Anagrams](https://leetcode.com/problems/anagrams/)

这道题目有两处疑问
 
- 字符数组转化为字符串
```
String s = String.valueOf(c);
//String s = c.toString();
```

- 是否建立新的`List<List<String>>`
```
//对于String[] strs = {"",""};得出的lists的大小是2
//why????
//List<List<String>> lists = new  ArrayList<List<String>>(map.values());
//for (String ss : map.keySet()) {
//		List<String> list = map.get(ss);
//		Collections.sort(list);
//		lists.add(list);
// }

for (String ss : map.keySet()) {
	Collections.sort(map.get(ss));
}
return new  ArrayList<List<String>>(map.values());
//return lists;

```


```
public static List<List<String>> groupAnagrams(String[] strs) {
	if(strs == null || strs.length == 0){
		return null;
	}
	
	//key是有序字母组成，list是具有相同字母组成的字符串
	HashMap<String, List<String>> map = new HashMap<String, List<String>>();
	for (int i = 0; i < strs.length; i++) {
		char[] c = strs[i].toCharArray();
		Arrays.sort(c);
		//TODO
		//toString得到每个都是新建的对象
		String s = String.valueOf(c);
//			String s = c.toString();
		//System.out.println(s.hashCode());
		if(map.containsKey(s)){
			map.get(s).add(strs[i]);
		}else{
			List<String> list = new ArrayList<String>();
			list.add(strs[i]);
			map.put(s, list);
		}
	}
	
	//TODO
	//对于String[] strs = {"",""};得出的lists的大小是2
	//why????
//		List<List<String>> lists = new  ArrayList<List<String>>(map.values());
//		for (String ss : map.keySet()) {
//			List<String> list = map.get(ss);
//			Collections.sort(list);
//			lists.add(list);
//		}
	
	for (String ss : map.keySet()) {
		Collections.sort(map.get(ss));
	}
	return new  ArrayList<List<String>>(map.values());
//		return lists;
	
}

```

