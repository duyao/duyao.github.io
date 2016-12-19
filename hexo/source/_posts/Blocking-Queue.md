title: Blocking Queue
toc: true
date: 2016-03-25 09:33:46
tags: java
categories:
---

## Introduction

A Queue that additionally supports operations that **wait for the queue to become non-empty when retrieving an element**, and **wait for space to become available in the queue when storing an element**.

BlockingQueue methods come in four forms, with different ways of handling operations that cannot be satisfied immediately, but may be satisfied at some point in the future: one throws an exception, the second returns a special value (either null or false, depending on the operation), the third blocks the current thread indefinitely until the operation can succeed, and the fourth blocks for only a given maximum time limit before giving up. These methods are summarized in the following table:


|Throws exception|	Special value	|Blocks	|Times out|
|:--:|:--:|:--:|:--:|
|Insert	|add(e)	|offer(e)|	put(e)|	offer(e, time, unit)|
|Remove|	remove()|	poll()|	take()|	poll(time, unit)|
|Examine|	element()|	peek()|	not applicable|	not applicable|

A BlockingQueue **does not accept null elements**. Implementations throw NullPointerException on attempts to add, put or offer a null. A null is used as a sentinel value to indicate failure of poll operations.

A BlockingQueue may be capacity bounded. At any given time it may have a remainingCapacity beyond which no additional elements can be put without blocking. A BlockingQueue without any intrinsic capacity constraints always reports a remaining capacity of Integer.MAX_VALUE.

BlockingQueue implementations are designed to be used primarily for **producer-consumer** queues, but additionally support the Collection interface. So, for example, it is possible to remove an arbitrary element from a queue using remove(x). However, such operations are in general not performed very efficiently, and are intended for only occasional use, such as when a queued message is cancelled.

BlockingQueue implementations are **thread-safe**. All queuing methods achieve their effects atomically using internal locks or other forms of concurrency control. However, the bulk Collection operations addAll, containsAll, retainAll and removeAll are not necessarily performed atomically unless specified otherwise in an implementation. So it is possible, for example, for addAll(c) to fail (throwing an exception) after adding only some of the elements in c.

A BlockingQueue does not intrinsically support any kind of "close" or "shutdown" operation to indicate that no more items will be added. The needs and usage of such features tend to be implementation-dependent. For example, a common tactic is for producers to insert special end-of-stream or poison objects, that are interpreted accordingly when taken by consumers.

Usage example, based on a typical producer-consumer scenario. Note that a BlockingQueue can safely be used with multiple producers and multiple consumers.

## Implementation

- ArrayBlockingQueue
- DelayQueue
- LinkedBlockingQueue
- PriorityBlockingQueue
- SynchronousQueue

## Example

该例子实现了一个在所给目录的所有文件中找关键字的方法
其本质是生产者和消费者问题，一个向队列中插入文件，另一个在该队列中取出文件
这个队列的实现是利用`BlockingQueue`做到取出为空时阻塞，添加满时阻塞
```
public static void main(String[] args) {
	Scanner in = new Scanner(System.in);
	System.out.println("Enter the file directory...");
	String directory = in.nextLine();
	System.out.println("Enter the keyword...");
	String keyword = in.nextLine();
	//10个队列
	BlockingQueue<File> queue = new ArrayBlockingQueue<File>(10);
	//1个生产者
	FileEnumrationTask task = new FileEnumrationTask(queue, new File(directory));
	new Thread(task).start();
	//100个消费者
	for (int i = 0; i < 100; i++) {
		new Thread(new SearchTask(queue, keyword)).start();
		
	}
}
```

文件遍历类是一个生产者，向队列中添加所给目录中的所有文件
```
class FileEnumrationTask implements Runnable{

	public static File DUMMY = new File("");
	private BlockingQueue<File> queue;
	private File startingDirectory;
	
	public FileEnumrationTask(BlockingQueue<File> queue, File startingDirectory) {
		this.queue = queue;
		this.startingDirectory = startingDirectory;
	}
	
	public void enumerate(File directory) throws InterruptedException{
		File[] files = directory.listFiles();
		for (File file : files) {
			if(file.isDirectory()){
				enumerate(file);
			}else{
				queue.put(file);
			}
			
		}
	}
	
	@Override
	public void run() {
		try {
			enumerate(startingDirectory);
			queue.put(DUMMY);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		
	}
	
}
```
单词查找类是消费者，负责都队列中取出文件，然后在该文件中找关键字
```
class SearchTask implements Runnable{
	private BlockingQueue<File> queue;
	private String keyword;

	public SearchTask(BlockingQueue<File> queue, String keyword) {
		this.queue = queue;
		this.keyword = keyword;
	}
	
	public void search(File file){
		try {
			Scanner in = new Scanner(file);
			int lineNum = 0;
			while(in.hasNextLine()){
				lineNum++;
				String line = in.nextLine();
				if(line.contains(keyword)){
					System.out.printf("%s : %d : %s%n",file.getPath(), lineNum, line);
				}
				
			}
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
		
	}

	@Override
	public void run() {
		boolean done = false;
		try {
			while(!done){
				File file = queue.take();
				if(file == FileEnumrationTask.DUMMY){
					queue.put(file);
					done = true;
				}else{
					search(file);
				}
				
			}
			
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		
	}
	
}

```
## Rerfences

https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/BlockingQueue.html
https://www.ibm.com/developerworks/cn/java/j-tiger06164/
