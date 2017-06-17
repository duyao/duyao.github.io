title: 332. Reconstruct Itinerary
date: 2016-03-07 21:30:07
tags: 
- leetcode
- algorithm
toc: true
---
[332. Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/)

## 思路
1. 要先将思维转换到图，每个点都可以看做是图上的一个点
2. 这不是dfs，因为不只是遍历，而是顺序遍历所有路径，即一笔画欧拉路径问题

## 参考
[wikipedia](https://en.wikipedia.org/wiki/Eulerian_path)

- Hierholzer's algorithm
[Eulerian Path and Circuit](http://www.graph-magics.com/articles/euler.php)


- Fleury's algorithm
[Don't burn your bridge](http://www.math.ku.edu/~jmartin/courses/math105-F11/Lectures/chapter5-part2.pdf)
[Fleury's algorithm](http://www.geeksforgeeks.org/fleurys-algorithm-for-printing-eulerian-path/)

## 理解
Hierholzer's algorithm
建立两个栈，一个是tmp暂时暂存元素栈，一个是final最终的结果栈
开始循环
从图中任一点出发，任意访问相邻得点，该节点放入tmp中
当一个节点周围没有相邻节点时，则将该节点从tmp移入到final中，设置当前结点为tmp的栈顶
当所有边均被访问，此过程结束
最后，将tmp放入final中
final的出栈顺序即为路径

还有本题目有个一个重要的数据结构`Map<String, PriorityQueue<String>> flights`
`PriorityQueue<String>`可以排序，而map记录每个点的边集

## 代码
递归法
```
Map<String, PriorityQueue<String>> flights;
LinkedList<String> path;

public List<String> findItinerary(String[][] tickets) {
	flights = new HashMap<>();
	path = new LinkedList<>();
	for (String[] ticket : tickets) {
		if(!flights.containsKey(ticket[0])){
			flights.put(ticket[0], new PriorityQueue<String>());
		}
		flights.get(ticket[0]).add(ticket[1]);
	}
	dfs("JFK");
	return path;
}

public void dfs(String departure) {
	PriorityQueue<String> arrivals = flights.get(departure);
	//访问所有的相邻节点
	while (arrivals != null && !arrivals.isEmpty())
		dfs(arrivals.poll());
	//注意是加到队头！
	path.addFirst(departure);
}
```
非递归
```
//这里的flights中只包含有邻居的节点
Map<String, PriorityQueue<String>> flights;
//返回值
LinkedList<String> path;

public List<String> findItinerary(String[][] tickets) {
	//使用优先队列来记录相邻接点且保证了字典顺序
	flights = new HashMap<String, PriorityQueue<String>>();
	path = new LinkedList<String>();
	if (tickets == null || tickets.length == 0) {
		return path;
	}
	for(String[] s : tickets){
		if(!flights.containsKey(s[0])){
			flights.put(s[0], new PriorityQueue<String>());
		}
		flights.get(s[0]).add(s[1]);
	}
	
	
	//http://www.graph-magics.com/articles/euler.php
	//设置一个退回栈就是circle，存放没有相邻接点的点
	//1.从开始节点遍历，将每个有相邻接点的点加入到结果集stack中
	//2.如果当前结点无相邻接点，则将其放入退回栈
	//3.然后在结果集中逆序选取有相邻接点的点作为当前结点
	//4.继续123，直到遍历过所有的边
	//5.最后将结果集并入到退回栈circle中
	Stack<String> circle = new Stack<String>();
	Stack<String> stack = new Stack<String>(); 
	String cur = "JFK";
	for(int i = 0; i < tickets.length; i++){
		while(!flights.containsKey(cur) || flights.get(cur).size() == 0){
			circle.add(cur);
			cur = stack.pop();
		}
		stack.add(cur);
		cur = flights.get(cur).poll();
	}
	//最后一个节点一定是没有邻接点的点，因此要放到退回栈circle中
	circle.add(cur);
	//结果集并入到退回栈中
	while(!stack.isEmpty()){
		circle.add(stack.pop());
	}
	//退回栈的逆序为最终路径
	while(!circle.isEmpty()){
		path.add(circle.pop());
	}
	return path;
}
```




