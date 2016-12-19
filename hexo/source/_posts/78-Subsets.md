title: 78. Subsets
toc: true
date: 2016-04-04 14:18:10
tags:
- leetcode
- algorithm
categories:
---
[78. Subsets](https://leetcode.com/problems/subsets/)
相似问题[46. Permutations](https://leetcode.com/problems/permutations/)
题解[46 Answer](http://duyao.github.io/2016/02/18/46-Permutations/)

## 方法一

### 思路

利用bitmap
所给数组是a[n]，则所有的方法总数是2^n的个数
所有的组合形式是0-2^n-1的二进制数

>
Number of subsets for {1 , 2 , 3 } = 2^3 . why ? 
case    possible outcomes for the set of subsets
  1   ->          Take or dont take = 2 
  2   ->          Take or dont take = 2  
  3   ->          Take or dont take = 2 
therefore , total = 2*2*2 = 2^3 = { { } , {1} , {2} , {3} , {1,2} , {1,3} , {2,3} , {1,2,3} }
Lets assign bits to each outcome  -> First bit to 1 , Second bit to 2 and third bit to 3
Take = 1
Dont take = 0
0) 0 0 0  -> Dont take 3 , Dont take 2 , Dont take 1 = { } 
1) 0 0 1  -> Dont take 3 , Dont take 2 ,   take 1       =  {1 } 
2) 0 1 0  -> Dont take 3 ,    take 2       , Dont take 1 = { 2 } 
3) 0 1 1  -> Dont take 3 ,    take 2       ,      take 1    = { 1 , 2 } 
4) 1 0 0  ->    take 3      , Dont take 2  , Dont take 1 = { 3 } 
5) 1 0 1  ->    take 3      , Dont take 2  ,     take 1     = { 1 , 3 } 
6) 1 1 0  ->    take 3      ,    take 2       , Dont take 1 = { 2 , 3 } 
7) 1 1 1  ->    take 3     ,      take 2     ,      take 1     = { 1 , 2 , 3 } 
In the above logic ,Insert S[i] only if (j>>i)&1 ==true   { j E { 0,1,2,3,4,5,6,7 }   i = ith element in the input array }
element 1 is inserted only into those places where 1st bit of j is 1 
   if( j >> 0 &1 )  ==> for above above eg. this is true for sl.no.( j )= 1 , 3 , 5 , 7 
element 2 is inserted only into those places where 2nd bit of j is 1 
   if( j >> 1 &1 )  == for above above eg. this is true for sl.no.( j ) = 2 , 3 , 6 , 7
element 3 is inserted only into those places where 3rd bit of j is 1 
   if( j >> 2 & 1 )  == for above above eg. this is true for sl.no.( j ) = 4 , 5 , 6 , 7 
Time complexity : O(n*2^n) , for every input element loop traverses the whole solution set length i.e. 2^n

reference:[bitmap solution](https://leetcode.com/discuss/9213/my-solution-using-bit-manipulation)

### 代码

```
public List<List<Integer>> subsets(int[] S) {
    Arrays.sort(S);
    //所有的结果数是2^n
    //<<是左移
    int totalNumber = 1 << S.length;
    List<List<Integer>> collection = new ArrayList<List<Integer>>(totalNumber);
    //结果所有数字的二进制表示
    for (int i=0; i<totalNumber; i++) {
        List<Integer> set = new LinkedList<Integer>();
        for (int j=0; j<S.length; j++) {
	        //System.out.println("i="+i+",j="+j+",i & 1<<j="+(i & (1<<j)));
            //对于每个i都把其二进制中的1添加进来
        	//因此把1不停的左移，就用&能筛选出所有的
        	if ((i & (1<<j)) != 0) {
                set.add(S[j]);
            }
        }
        collection.add(set);
    }
    return collection;
}
```

## 方法二
### 思路
递归遍历，有点像dfs

subsets = {1 , 2 , 3 }
添加顺序为
{}
{1}
{1,2}
{1,2,3}
{1,3}//path.remove(path.size() - 1),此时i=3,退回i=2,添加3
{2}
{2,3}
{3}

### 代码
```
public List<List<Integer>> subsets(int[] nums) {

	List<List<Integer>> res = new ArrayList<List<Integer>>();
	res.add(new ArrayList<Integer>());
	//添加时保证有序
	Arrays.sort(nums);
	f(nums, 0, new ArrayList<Integer>(), res);
	return res;
}

public void f(int[] nums, int index, List<Integer> path,
		List<List<Integer>> res) {
	
	for (int i = index; i < nums.length; i++) {
		path.add(nums[i]);
		//这里添加一定要new一个，不然后面path删除元素会影响结果集
		//res.add(path);
		res.add(new ArrayList<Integer>(path));
		f(nums, i + 1, path, res);
		path.remove(path.size() - 1);
	}
}
```
## 方法三

### 思路
循环遍历
1. 在结果集中添加一个空元素
2. 遍历每个数字
2.1 ，遍历所有结果集中的结果,将这个数字添加到每个结果中
2.2 更新结果集

subsets = {1 , 2 , 3 }
添加顺序为
{} //add empty
{1} // add 1to {}, add 1 for ends
{2} //add to {},{1}
{1,2} //add 2 for ends
{3} // add 3 to {}, {1}, {2}, {1,2}
{1,3}
{2,3}
{1,2,3} //add 3 for ends

### 代码

- 错误一

java.util.ConcurrentModificationException
```
for (Integer i : nums) {
	// java.util.ConcurrentModificationException
	List<List<Integer>> tmp = new ArrayList<List<Integer>>(res);
	for (List<Integer> sub : tmp) {
		List<Integer> tt = new ArrayList<Integer>(sub);
		tt.add(i);
		tmp.add(tt);
	}
	res = tmp;
}
```

- 错误二

添加关联影响
```
for (Integer i : nums) {
	List<List<Integer>> tmp = new ArrayList<List<Integer>>();
	for (List<Integer> sub : res) {
		//这里sub添加后，res会马上改变
		sub.add(i);
		tmp.add(sub);
	}
	res.addAll(tmp);
}
```

- 正确代码

```
public List<List<Integer>> subsets1(int[] nums){
	List<List<Integer>> res = new ArrayList<List<Integer>>();
	res.add(new ArrayList<Integer>());
	Arrays.sort(nums);
	for (Integer i : nums) {			
		//记录本次循环添加的list,一定与res无关，不然ConcurrentModificationException
		List<List<Integer>> tmp = new ArrayList<List<Integer>>();
		for (List<Integer> sub : res) {
			//对于sub建立副本，不然修改sub，res会改变
			List<Integer> list = new ArrayList<Integer>(sub);
			list.add(i);
			tmp.add(list);
		}
		//同步回res
		res.addAll(tmp);		
	}
	return res;
}
```