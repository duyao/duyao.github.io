title: 133. Clone Graph
date: 2016-02-16 16:31:06
tags:
- leetcode
- algorithm
---

[133. Clone Graph](https://leetcode.com/problems/clone-graph/)

该题目是通过图的遍历复制图

注意的是返回的每个结点必须是新的结点，而不是原来结点的指针

```
class UndirectedGraphNode {
	int label;
	List<UndirectedGraphNode> neighbors;

	UndirectedGraphNode(int x) {
		label = x;
		neighbors = new ArrayList<UndirectedGraphNode>();
	}
};

// dfs中是否访问过的标记
HashMap<Integer, UndirectedGraphNode> map = new HashMap<Integer, UndirectedGraphNode>();

public UndirectedGraphNode cloneGraph(UndirectedGraphNode node) {
	if (node == null) {
		return null;
	}

	if (map.containsKey(node.label)) {
		return map.get(node.label);
	}

	UndirectedGraphNode clone = new UndirectedGraphNode(node.label);
	//这里放入的节点一定是clone
	//因为本题目是clone图，递归返回的值是从map中获取，而获取到的值必须是一个新节点
	//map.put(node.label, node);
	map.put(clone.label, clone);
	for (UndirectedGraphNode next : node.neighbors) {
		
		//这里添加相邻接点必须是新构造的节点，而不是原来的
		//clone.neighbors.add(next);
		//递归的返回值是新的结点
		clone.neighbors.add(cloneGraph(next));
	}
	return clone;

}
```
