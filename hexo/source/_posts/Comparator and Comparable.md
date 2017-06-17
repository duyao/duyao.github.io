title: Comparator and Comparable
toc: true
date: 2016-03-21 12:55:47
tags: java
categories:
---

## 比较

在集合框架中有两种比较接口： Comparable 接口和 Comparator 接口。
Comparable 是通用的接口，用户可以实现它来完成自己特定的比较，在java.lang下
Comparator 可以看成一种算法的实现，在需要容器集合实现比较功能的时候，来指定这个比较器，这可以看成一种设计模式，将算法和数据分离，在java.util下。

前者应该比较固定，和一个具体类相绑定，而后者比较灵活，它可以被用于各个需要比较功能的类使用。

一个类实现了 Camparable 接口表明这个类的对象之间是可以相互比较的。如果用数学语言描述的话就是这个类的对象组成的集合中存在一个全序。这样，这个类对象组成的集合就可以使用 Sort 方法排序了。

而 Comparator 的作用有两个，体现了**策略模式**，就是不改变对象自身，而用一个策略对象（strategy object）来改变它的行为：
1 、如果类的设计师没有考虑到 Compare 的问题而没有实现 Comparable 接口，可以通过 Comparator 来实现比较算法进行排序；
2 、为了使用不同的排序标准做准备，比如：升序、降序或其他什么序。

## Comparator

我们如果需要控制某个类的次序，而该类本身不支持排序（即没有实现Comparable接口）；那么可以建立一个该类的比较器来排序，这个比较器只需要实现Comparator接口即可。

通过实现Comparator类来新建一个比较器，然后通过该比较器来对类进行排序。Comparator 接口其实就是一种策略模式的实践

若一个类要实现Comparator接口：它一定要实现`compareTo(T o1, T o2) `函数，但可以不实现 `equals(Object obj)` 函数

## Comparatable

若一个类实现了Comparable接口，就意味着“该类支持排序”。  即然实现Comparable接口的类支持排序，假设现在存在“实现Comparable接口的类的对象的List列表(或数组)”，则该List列表(或数组)可以通过 Collections.sort（或 Arrays.sort）进行排序。

此外，“实现Comparable接口的类的对象”可以用作“有序映射(如TreeMap)”中的键或“有序集合(TreeSet)”中的元素，而不需要指定比较器。

假设我们通过 x.compareTo(y) 来“比较x和y的大小”。若返回“负数”，意味着“x比y小”；返回“零”，意味着“x等于y”；返回“正数”，意味着“x大于y”。

|类型|排序方法|
|---|--|
|BigDecimal,BigInteger,Byte,Double, Float,Integer,Long,Short|按数字大小排序|
|Character|按 Unicode 值的数字大小排序|
|String|按字符串中字符 Unicode 值排序|

## 例子

- Item.java

```
public class Item implements Comparable<Item>{

	private String description;
	private int partNum;

	public Item(String des, int num){
		this.description = des;
		this.partNum = num;
	}

	public String getDescription(){
		return this.description;
	}

	public int getPartNum(){
		return this.partNum;
	}

	@Override
	public String toString() {
		return "[" + description + "," + partNum + "]";
	}

//	@Override
//	public int hashCode() {
//		return Objects.hash(description, partNum);
//	}


//	@Override
//	public boolean equals(Object obj) {
//		if(this == obj) return true;
//		if(obj == null) return false;
//		if(this.getClass() != obj.getClass()) return false;
//		
//		Item i = (Item) obj;
//		return Objects.equals(this.description, i.description) && this.partNum == i.partNum;
//	}


	@Override
	public int compareTo(Item o) {
		return Integer.compare(partNum, o.partNum);
	}



}
```

- Main

```
public static void main(String[] args) {

	SortedSet<Item> mySet = new TreeSet();
	//Item中是按照partNum进行比较的
	mySet.add(new Item("To", 9234));
	mySet.add(new Item("Wi", 362));
	mySet.add(new Item("Mo", 1912));
	mySet.add(new Item("Mo", 12));
	mySet.add(new Item("Mo", 342));
	System.out.println(mySet);

	//使用comparator，类Item可以没有实现comparable
	//新建TreeSet通过传入比较器comparator来完成
	SortedSet<Item> setByDes = new TreeSet<Item>(new Comparator<Item>() {

		@Override
		public int compare(Item o1, Item o2) {
			//先按照des排序，对于相等的des则按照partNum排序
			String aDes = o1.getDescription();
			String bDes = o2.getDescription();
			if(aDes.compareTo(bDes) > 0){
				return 1;
			}else if(aDes.compareTo(bDes) < 0){
				return -1;
			}else{
				//注意前面有符号-，表示逆序，从大到小的顺序
				return -Integer.compare(o1.getPartNum(), o2.getPartNum());
			}
			//按照des排序，对于des相同，则只显示最小的partNum
//				return aDes.compareTo(bDes);
		}

	});

//		setByDes.addAll(mySet);
	setByDes.add(new Item("To", 9234));
	setByDes.add(new Item("Wi", 362));
	setByDes.add(new Item("Mo", 1912));
	setByDes.add(new Item("Mo", 12));
	setByDes.add(new Item("Mo", 342));
	System.out.println(setByDes);
}
```


http://blog.csdn.net/mageshuai/article/details/3849143
http://www.cnblogs.com/sunflower627/p/3158042.html
