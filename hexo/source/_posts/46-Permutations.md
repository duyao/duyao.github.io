title: 46. Permutations
date: 2016-02-18 17:47:31
tags: leetcode
categories: algorithm
---
[46. Permutations](https://leetcode.com/problems/permutations/)
相似问题[78. Subsets](https://leetcode.com/problems/subsets/)
题解[78 Answer](http://duyao.github.io/2016/04/04/78-Subsets/)


回溯法backtrack

递归和非递归

```
//递归
public static List<List<Integer>> permute(int[] nums) {
	List<List<Integer>> lists = new ArrayList<List<Integer>>();
	backtrack(nums, 0, lists, new ArrayList<Integer>());
	return lists;
}


public static void backtrack(int[] nums, int i,List<List<Integer>> lists,List<Integer> curList){
	//个数满足，就可以添加到答案中
	if(curList.size() == nums.length){
		lists.add(curList);
		//返回到上一层递归
		return ;
	}

	//注意循环条件是j <= curList.size()
	//这样才能将新的数字添加到所有位置(有等于号)，且一直是对同一list做修改
	for(int j = 0; j <= curList.size(); j++){
		//获取到最新修改的list
		List<Integer> newList = new ArrayList<Integer>(curList);
		newList.add(j, nums[i]);
		System.out.println("add "+"nums[i="+i+"]="+nums[i]+" at pos j="+j);
		System.out.println("newList : ");
		for (Integer integer : newList) {
			System.out.print(integer + " ");
		}
		System.out.println();
		backtrack(nums, i+1, lists, newList);
	}


}


//非递归
public static List<List<Integer>> permute(int[] nums) {
	List<List<Integer>> list = new ArrayList<List<Integer>>();
	if(nums == null  || nums.length == 0){
		return list;
	}
	//先添加第一个元素，并将list加入到结果集中
	List<Integer> ll = new ArrayList<Integer>();
	ll.add(nums[0]);
	list.add(ll);
	//对于所有元素进行添加
	//第一个元素已经添加，所以从1开始
	for(int i = 1; i < nums.length; i++){

		List<List<Integer>> newList = new ArrayList<List<Integer>>();
		//枚举所有的位置
		//j <= i表示位置始终与元素个数相关
		for(int j = 0; j <= i; j++){
			//对于所有已经存在的list，尝试添加所有位置
			for (List<Integer>  tmp : list) {
				List<Integer> newLL = new ArrayList<Integer>(tmp);
				newLL.add(j,nums[i]);
				newList.add(newLL);
			}
		}
		//更新list，使得刚刚做的修改生效
		list = newList;
	}
	return list;
}
```
