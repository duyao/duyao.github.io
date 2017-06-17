title: Callable and Future
toc: true
date: 2016-03-26 15:08:55
tags: java
categories:
---

## Callable 
与`Runnable`相似，只是`Runnable`无返回值
A task that returns a result and may throw an exception. Implementors define a single method with no arguments called `call`.
The Callable interface is similar to Runnable, in that both are designed for classes whose instances are potentially executed by another thread. A Runnable, however, does not return a result and cannot throw a checked exception.

## Future
异步保存结果
A Future represents the result of an asynchronous computation. 
Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation. 
The result can only be retrieved using method `get` when the computation has completed, blocking if necessary until it is ready. 
Cancellation is performed by the `cancel` method. 
Additional methods are provided to determine if the task completed normally or was cancelled. 
Once a computation has completed, the computation cannot be cancelled. If you would like to use a Future for the sake of cancellability but not provide a usable result, you can declare types of the form Future<?> and return null as a result of the underlying task.

## 例子
实现了在制定文件目录中的所有文件查找关键字的个数

- 计数任务

```
class MatchCounter implements Callable<Integer>{
	private File directory;
	private String keyWord;
	private int count;
	
	MatchCounter(File directory, String keyWord){
		this.directory = directory;
		this.keyWord = keyWord;
	}
	
	//在文件中搜索关键字
	public boolean search(File file){
		try {
			Scanner in = new Scanner(file);
			boolean found = false;
			while(!found && in.hasNextLine()){
				String line = in.nextLine();
				if(line.contains(keyWord)){
					found = true;
				}
			}
			return found;
		} catch (FileNotFoundException e) {
			e.printStackTrace();
			return false;
		}
		
		
	}

	@Override
	public Integer call() throws Exception {
		count = 0;
		File[] files = directory.listFiles();
		//设置多个线程来访问文件，放在结果集中
		List<Future<Integer>> results = new ArrayList<Future<Integer>>();
		
		for (File file : files) {
			if(file.isDirectory()){
				//是目录，开启一个计算任务
				MatchCounter counter = new MatchCounter(file, keyWord);
				//可以存放多个Callable
				FutureTask<Integer> task = new FutureTask<Integer>(counter);
				//新的任务开启并放在结果集中
				results.add(task);
				Thread t = new Thread(task);
				t.start();
			}else{
				//是文件，搜索关键字，计数
				if(search(file)){
					count++;
				}
				
			}
		}
		
		//对于结果集中的所有结果，计算所有的次数
		for (Future<Integer> future : results) {
			//如果有些线程没有结果，就阻塞
			count += future.get();
		}
		
		return count;
	}
	
	
} 
```

- Main

```
public static void main(String[] args) {
	Scanner in = new Scanner(System.in);
	System.out.println("Enter the directory...");
	String directory = in.nextLine();
	System.out.println("Enter the keyword...");
	String keyword = in.nextLine();
	
	MatchCounter counter = new MatchCounter(new File(directory), keyword);
	FutureTask<Integer> task = new FutureTask<Integer>(counter);
	Thread t = new Thread(task);
	t.start();
	
	try {
		System.out.println(task.get() + " matching files.");
	} catch (InterruptedException e) {
		e.printStackTrace();
	} catch (ExecutionException e) {
		e.printStackTrace();
	}
	
	
}
```
