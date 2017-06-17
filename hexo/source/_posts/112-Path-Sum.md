title: 112. Path Sum
toc: true
date: 2017-02-16 21:25:30
tags:
- leetcode
- algorithm
categories:

---
## [112. Path Sum](https://leetcode.com/problems/path-sum/?tab=Solutions)

此题目是树的遍历

由于树的遍历采用的递归方式，因此在遍历过程中可以记录很多信息，但是要注意递归过程中返回上一层时，记录的基本类型的值都会变为原来的

```
public boolean hasPathSum(TreeNode root, int sum) {
    if(root == null){
        return false;
    }else{
        if(root.left == null && root.right ==null && sum - root.val == 0){
            return true;
        }

        //这里是或运算
        //因为每一层递归都会记录状态，同时返回时状态会消失
        return hasPathSum(root.left, sum-root.val) || hasPathSum(root.right, sum - root.val);
    }

}
```
## [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/?tab=Description)


```
public List<List<Integer>> pathSum(TreeNode root, int sum) {

  List<List<Integer>> res = new ArrayList<>();
  visit(root, new ArrayList<>(), sum, res);
  return  res;
}
public void visit(TreeNode root, List<Integer> list,  int sum, List<List<Integer>> res) {
  if(root != null){
      list.add(root.val);

      if(root.left == null && root.right == null && sum - root.val == 0 ){
          res.add(new ArrayList<>(list));
      }


      visit(root.left, list, sum - root.val, res);
      visit(root.right, list, sum - root.val, res);

      list.remove(list.size()-1);

  }

}
```


## [129. Sum Root to Leaf Numbers](https://leetcode.com/problems/sum-root-to-leaf-numbers/)

值传递的递归
```
public int sumNumbers(TreeNode root) {
	if (root == null) {
		return 0;
	}
	int sum = cal(root, new StringBuffer());
	return sum;
}

public int cal(TreeNode cur, StringBuffer sb) {

	sb.append(cur.val);
	if (cur.left == null && cur.right == null) {
		int tmp = Integer.valueOf(sb.toString());
		sb.deleteCharAt(sb.length()-1);
		return tmp;
	} else {

		int sum = 0;
		if (cur.left != null) {
			sum += cal(cur.left, sb);
		}
		if (cur.right != null) {
			sum += cal(cur.right, sb);
		}
		sb.deleteCharAt(sb.length()-1);
		return sum;
	}
}
```

非值传递
```
public int sumNumbers(TreeNode root) {

    List<Integer> res = new ArrayList<>();
    visit(root, new StringBuffer(), res);
    int sum = 0;
    for (Integer i: res
         ) {
        sum += i;

    }
    return sum;


}
public void visit(TreeNode root, StringBuffer sb, List<Integer> res){
    if(root != null){
        sb.append(root.val);

        if(root.left == null && root.right == null){
            res.add(Integer.valueOf(sb.toString()));

        }


        visit(root.left, sb, res);
        visit(root.right, sb, res);

        sb.deleteCharAt(sb.length()-1);


    }
}
```
