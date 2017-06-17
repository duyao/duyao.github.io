title: juc之线程池
toc: true
date: 2017-03-05 16:40:21
tags: jvm
categories: note
---

# 基本使用
![比较图](http://7xilc8.com1.z0.glb.clouddn.com/executor.png)
Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；
然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；
抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；
然后ThreadPoolExecutor继承了类AbstractExecutorService。

## 线程池的种类
我们可以通过ThreadPoolExecutor来创建一个线程池。
```java
new  ThreadPoolExecutor(int corePoolSize, int maximumPoolSize,
  long keepAliveTime, TimeUnit unit,
  BlockingQueue<Runnable> workQueue,
  ThreadFactory threadFactory,
  RejectedExecutionHandler handler);
```
创建一个线程池需要输入几个参数：
- corePoolSize（线程池的基本大小）
当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。

- runnableTaskQueue（任务队列）
用于保存等待执行的任务的阻塞队列。 可以选择以下几个阻塞队列。
  - ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列
  此队列按 FIFO（先进先出）原则对元素进行排序。
  - LinkedBlockingQueue：一个基于链表结构的阻塞队列
  此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
  - SynchronousQueue：一个不存储元素的阻塞队列
  每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
  - PriorityBlockingQueue：一个具有优先级的无限阻塞队列。

直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在**处理可能具有内部依赖性的请求集时出现锁**。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

它将任务直接提交给线程而不保存它们。在此，如果不存在可用于立即运行任务的线程，
则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。
直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。
当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。  
SynchronousQueue线程安全的Queue，可以存放若干任务（但当前只允许有且只有一个任务在等待），其中每个插入操作必须等待另一个线程的对应移除操作，也就是说A任务进入队列，B任务必须等A任务被移除之后才能进入队列，否则执行异常策略。
你来一个我扔一个，所以说SynchronousQueue没有任何内部容量。

比如：核心线程数为2，最大线程数为3；使用SynchronousQueue。
当前有2个核心线程在运行，又来了个A任务，两个核心线程没有执行完当前任务，根据如果运行的线程等于或多于 corePoolSize，
则 Executor 始终首选将请求加入队列，而不添加新的线程。所以A任务被添加到队列，此时的队列是SynchronousQueue，
当前不存在可用于立即运行任务的线程，因此会构造一个新的线程，此时又来了个B任务，两个核心线程还没有执行完。
新创建的线程正在执行A任务，所以B任务进入Queue后，最大线程数为3，发现没地方仍了。就只能执行异常策略(RejectedExecutionException)。

无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

有界队列。当使用有限的 maximumPoolSizes时，有界队列（如 ArrayBlockingQueue）**有助于防止资源耗尽，但是可能较难调整和控制**。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。  

- maximumPoolSize（线程池最大大小）
线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。

- ThreadFactory
用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。

- RejectedExecutionHandler（饱和策略）
当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。
  - AbortPolicy：直接抛出异常。
  - CallerRunsPolicy：只用调用者所在线程来运行任务。
  - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
  - DiscardPolicy：不处理，丢弃掉。
当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。

- keepAliveTime（线程活动保持时间）
线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。

- TimeUnit（线程活动保持时间的单位）
可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

线程池可以分为以下几个种类
- 固定大小线程池newFixedThreadPool
超过最大数量就会加入到LinkedBlockingQueue，这个队列的数量是没有限制的，因此如果一直插入就会消耗内存
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
  return new ThreadPoolExecutor(nThreads, nThreads,
                               0L, TimeUnit.MILLISECONDS,
                               new LinkedBlockingQueue<Runnable>());
}
```
- newSingleThreadExecutor
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行：
```java
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}
```
- newCachedThreadPool
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
CachedThreadPool中线程实例默认超时时间为60s，超过这个时间，线程实例停止并被移出CachedThreadPool，适用于生存期短、异步的线程任务。
```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```
- newScheduledThreadPool
创建一个定长线程池，支持定时及周期性任务执行。

## 状态
当创建线程池后，初始时，线程池处于RUNNING状态；
如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；
如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；
当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态
## 原理
```java
//任务缓存队列，用来存放等待执行的任务
private final BlockingQueue<Runnable> workQueue;  
//线程池的主要状态锁，对线程池状态（比如线程池大小、runState等）的改变都要使用这个锁            
private final ReentrantLock mainLock = new ReentrantLock();
//用来存放工作集
private final HashSet<Worker> workers = new HashSet<Worker>();  

private volatile long  keepAliveTime;    //线程存货时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
private volatile int   poolSize;       //线程池中当前的线程数
private volatile RejectedExecutionHandler handler; //任务拒绝策略
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
private long completedTaskCount;   //用来记录已经执行完毕的任务个数
```

### 执行规则

这规则主要体现在execute方法中，里面有很多if语句的判断：

1）如果当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会创建一个线程去执行这个任务；

2）如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，
若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；

3）如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；

4）如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，直至线程池中的线程数目不大于corePoolSize；

5）如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过keepAliveTime，线程也会被终止。

同时线程池中的线程主要靠workers这个set来管理线程，添加删除等操作都需要获得锁mainLock。
提交的线程都是被封装后的`Worker`，其中有一个`runWorker()`的方法就是线程执行的过程，里面会调用`afterExecute`等方法，实现自己的策略。

**当前线程运行完后，再到workQueue阻塞队列中去获取一个task出来，继续运行，这样就保证了线程池中有一定的线程一直在运行；**
此时若跳出了while循环，只有workQueue队列为空才会出现或出现了类似于shutdown的操作，自然运行队列会减少1，当再有新的线程进来的时候，就又开始向worker里面放数据了，这样以此类推，实现了线程池的功能。

同时需要注意的是在线程池中实现线程的使用run方法启动启动线程，因为run方法直接调用不会启动新的线程，也是因为这样，导致了你无法获取到你自己的线程的状态，因为线程池是直接调用的run方法，而不是start方法来运行。

这里有一个非常巧妙的设计方式，假如我们来设计线程池，可能会有一个任务分派线程，当发现有线程空闲时，就从任务缓存队列中取一个任务交给空闲线程执行。但是在这里，并没有采用这样的方式，因为这样会要额外地对任务分派线程进行管理，无形地会增加难度和复杂度，这里直接让执行完任务的线程去任务缓存队列里面取任务来执行。
### 线程池监控
通过线程池提供的参数进行监控。线程池里有一些属性在监控线程池的时候可以使用

taskCount：线程池需要执行的任务数量。
completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount。
largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。
getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不减。
getActiveCount：获取活动的线程数。

# 扩展和增强线程池
在线程池中，如果一个线程抛异常，并不会影响其他线程的执行，这个线程会被停止，同时该线程池也不会报错，只是结果输出的时候少一个。
这时候可以使用future.get()每一个结果。这样就能找出错误来。
或者可以使用回调接口增强线程池。
## 回调接口
- beforeExecute
- afterExecute
- terminated


通过继承线程池并重写线程池的beforeExecute，afterExecute和terminated方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。
如监控任务的平均执行时间，最大执行时间和最小执行时间等。
```java
class ExtendedExecutor extends ThreadPoolExecutor {
    // ...
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        if (t == null && r instanceof Future<?>) {
          try {
            Object result = ((Future<?>) r).get();
          } catch (CancellationException ce) {
              t = ce;
          } catch (ExecutionException ee) {
              t = ee.getCause();
          } catch (InterruptedException ie) {
              Thread.currentThread().interrupt(); // ignore/reset
          }
        }
        if (t != null)
          System.out.println(t);
    }
}
```

## 拒绝策略

## 自定义ThreadFactory
使用`new ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
           BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);`传入一个ThreadFactory，实现方法
比如
```java
ExecutorService ctp =  Executors.newCachedThreadPool(new ThreadFactory() {  
  private AtomicInteger count = new AtomicInteger();  
  public Thread newThread(Runnable r) {  
      int c = count.incrementAndGet();  
      System.out.println("create no " + c + " Threads");  
      return new WorkThread(r,count);  

      }  
});
```
ThreadFactory是一个接口
```java
public interface ThreadFactory {

    /**
     * Constructs a new {@code Thread}.  Implementations may also initialize
     * priority, name, daemon status, {@code ThreadGroup}, etc.
     *
     * @param r a runnable to be executed by new thread instance
     * @return constructed thread, or {@code null} if the request to
     *         create a thread is rejected
     */
    Thread newThread(Runnable r);
}
```

## 线程池数量的选择
线程池的大小需要考虑cpu数量、内存大小等因素。
Ncpu = cpu数量
Ucpu = 目标cpu的使用率，0<=Ucpu<=1
W/C = 等待时间与计算时间的比率
为了保持处理器达到期望的使用率，最优的池大小等于Nthread = Ncpu \* Ucpu \* (1 + W\/C)
我们可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数。

要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：
任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
任务的优先级：高，中和低。
任务的执行时间：长，中和短。
任务的依赖性：是否依赖其他系统资源，如数据库连接。

任务性质不同的任务可以用不同规模的线程池分开处理。
- CPU密集型任务配置尽可能小的线程，如配置Ncpu+1个线程的线程池。
- IO密集型任务则由于线程并不是一直在执行任务，则配置尽可能多的线程，如2*Ncpu。
- 混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。
- 优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。
- 执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。
- 依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。


建议使用有界队列，有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点，比如几千。有一次我们组使用的后台任务线程池的队列和线程池全满了，不断的抛出抛弃任务的异常，通过排查发现是数据库出现了问题，导致执行SQL变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞住，任务积压在线程池里。如果当时我们设置成无界队列，线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。当然我们的系统所有的任务是用的单独的服务器部署的，而我们使用不同规模的线程池跑不同类型的任务，但是出现这样问题时也会影响到其他任务。
http://www.infoq.com/cn/articles/java-threadPool

# ForkJoin

其思想是分而治之
底层使用双端队列(就是数组)实现，使用的是`unsafe`中的无锁CAS机制，所以有很锁算地址的操作
同时将很多变量放在一个int中表示，这样可以保证数值之间的一致性，另一个方面就是CAS操作仅仅是面向一个变量的，多个变量是不支持CAS操作的
![比较图](http://7xilc8.com1.z0.glb.clouddn.com/forkjoin.png)

## 工作窃取（work-stealing）
工作窃取算法是指某个线程从其他队列里窃取任务来执行。工作窃取的运行流程
![比较图](http://7xilc8.com1.z0.glb.clouddn.com/worksteal.png)
那么为什么需要使用工作窃取算法呢？假如我们需要做一个比较大的任务，我们可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，于是把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应，比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。

## ForkJoin计算过程

第一步分割任务。首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。

第二步执行任务并合并结果。分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。

Fork/Join使用两个类来完成以上两件事情：

- ForkJoinTask
我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
    - RecursiveAction：用于没有返回结果的任务。
    - RecursiveTask ：用于有返回结果的任务。

- ForkJoinPool
ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。

## 例子
```java
public class ForkJoinDemo extends RecursiveTask<Long> {
    private static final int THREADHOLD = 10000;
    private long start;
    private long end;

    public ForkJoinDemo(long start, long end) {
        this.start = start;
        this.end = end;
    }


    @Override
    protected Long compute() {
        long sum = 0;
        boolean canCompute = (end - start) < THREADHOLD;
        //分为可以执行和不能执行
        if (canCompute) {
            for (long i = start; i < end; i++) {
                sum += i;
            }
        } else {
            //分成100个小任务
            long step = (start + end) / 100;
            ArrayList<ForkJoinDemo> subTasks = new ArrayList<>();
            long pos = start;
            for (int i = 0; i < 100; i++) {
                long lastOne = pos + step;
                //最后一个任务不够100
                if (lastOne > end) {
                    lastOne = end;
                }
                ForkJoinDemo subTask = new ForkJoinDemo(pos, lastOne);
                pos += step+1;
                subTasks.add(subTask);
                //提交到大任务
                subTask.fork();
            }
            for (ForkJoinDemo t : subTasks) {
                //等待
                sum += t.join();
            }
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinDemo task = new ForkJoinDemo(0, 200000L);

        ForkJoinTask<Long> result = forkJoinPool.submit(task);
        try {
            //会抛出异常
            long res = result.get();
            System.out.println("sum = " + res);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

http://www.infoq.com/cn/articles/fork-join-introduction
