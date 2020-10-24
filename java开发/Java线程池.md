### 线程池的优势

1. 降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；
2. 提高系统响应速度，当有任务达到时，通过复用已存在的线程，无需等待新线程的创建便能立即执行；
3. 方便线程并发数的管控。线程如果无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu的过度切换（cpu切换线程是有时间成本的）；
4. 提供更强大的功能，延时定时线程池。

### 线程池的基本参数

**corePoolSize**：

线程池的基本大小，即在没有任务需要执行的时候线程池的大小，并且只有在工作队列满了的时候才会创建超出这个数量的线程。这里需要注意的是：在刚刚创建ThreadPoolExecutor的时候，线程并不会立即启动，而是要等到有任务提交时才会启动，除非调用了prestartCoreThread/prestartAllCoreThreads事先启动核心线程。再考虑到keepAliveTime和allowCoreThreadTimeOut超时参数的影响，所以没有任务需要执行的时候，线程池的大小不一定是corePoolSize。

**maximumPoolSize**：

**线程池中允许的最大线程数**，线程池中的当前线程数目不会超过该值。如果队列中任务已满，并且当前线程个数小于maximumPoolSize，那么会创建新的线程来执行任务。这里值得一提的是largestPoolSize，该变量记录了线程池在整个生命周期中曾经出现的最大线程个数。为什么说是曾经呢？因为线程池创建之后，可以调用setMaximumPoolSize()改变运行的最大线程的数目。

**poolSize**：

线程池中**当前**线程的数量，当该值为0的时候，意味着没有任何线程，线程池会终止；同一时刻，poolSize不会超过maximumPoolSize。

**那么poolSize、corePoolSize、maximumPoolSize三者的关系是如何的呢？**

当新提交一个任务时：

（1）如果poolSize<corePoolSize，新增加一个线程处理新的任务。

（2）如果poolSize=corePoolSize，新任务会被放入阻塞队列等待。

（3）如果阻塞队列的容量达到上限，且这时poolSize<maximumPoolSize，新增线程来处理任务。

（4）如果阻塞队列满了，且poolSize=maximumPoolSize，那么线程池已经达到极限，会根据饱和策略RejectedExecutionHandler拒绝新的任务。

**keepAliveTime：**

如果一个线程处在空闲状态的时间超过了该属性值，就会因为超时而退出。是否允许超时退出则取决于上面的逻辑。

### ThreadPoolExecutor

参数信息：

| 序号 | 名称            | 类型                      | 含义             |
| ---- | --------------- | ------------------------- | ---------------- |
| 1    | corePoolSize    | int                       | 核心线程池大小   |
| 2    | maximumPoolSize | int                       | 最大线程池大小   |
| 3    | keepAliveTime   | long                      | 线程最大空闲时间 |
| 4    | unit            | TimeUnit                  | 时间单位         |
| 5    | workQueue       | BlockingQueue\<Runnable\> | 线程等待队列     |
| 6    | threadFactory   | ThreadFactory             | 线程创建工厂     |
| 7    | handler         | RejectedExecutionHandler  | 拒绝策略         |

通过该类构造符合我们期待的线程池

```Java
// ThreadPoolExecutor的构造器
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
    // defaultThreadFactory：executor创建新线程时使用的默认工厂
    // defaultHandler：当达到最大线程数和队列满后的处理器，默认抛出RejectedExecutionException
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
}

// 默认线程工厂，是Executors中的静态内部类
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    // ThreadGroup和Thread的关系相当于集合和元素的关系，ThreadGroup之间的关系是树的关系。main方法执行后，将自动创建system线程组和main线程组，main方法所在线程存放在main线程组中。
    private final ThreadGroup group; 
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
        	Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                      "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                        namePrefix + threadNumber.getAndIncrement(),
                        0);
        // 关闭守护线程
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            // 设为普通优先级
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

### 四种线程池

#### CachedThreadPool

```Java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

// 构造器
public static ExecutorService newCachedThreadPool() {
    // corePoolSize: 线程池基本大小，这里设为0
    // maximumPoolSize: 线程池允许存在的最大线程数量，这里设为最大整数
    // keepAliveTime: 当线程数大于corePoolSize，空闲线程的存在时间，超过该时间后空闲线程将会退出，这里设为60秒
    // unit: keepAliveTime的时间单元
    // workQueue：在提交任务被执行前存储任务的队列，这里采用同步队列
    // threadFactory: 线程创建工厂，这里使用默认线程工厂
    // handler: 拒绝任务执行处理器，默认为任务拒绝处理器
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

返回ThreadPoolExecutor实例，corePoolSize为0，maximumPoolSize为Integer.MAX_Value，keepAliveTime为60L，workQueue为SynchronousQueues（同步队列）。SyschronousQueue的特点是入队出队必须同时传递，因为CachedThreadPool线程创建没有限制（最大整数），不会有任务在队列中等待

**特点总结：**

1. 可以无限扩大的线程池
2. 适合处理执行事件比较小的任务
3. 线程空闲时间60秒后被杀死，当长时间处于空闲状态的时候，线程池几乎不占用资源
4. 同步队列没有存储空间，只要有请求到来，就必须找到一条空闲线程去处理这个请求，找不到就新开一个线程

**问题：**

如果出现成提交任务的速度远远大于线程池处理速度，CachedThreadPool会不断地创建新线程来执行任务，这样可能会导致系统耗尽CPU和内存资源，**所以在使用该线程池是，一定要注意控制并发的任务数，否则创建大量的线程可能导致严重的性能问题**

适用场景：快速处理大量耗时较短的(任务数量不能太大)任务，比如Netty的NIO接收请求。

##### SynchronousQueue

S'y'n'ch'ronousQueue中有抽象静态内部类Transfer\<E\>，有两个实现类TransferStack\<E\>和TransferQueue\<E\>，Transfer\<E\>使用抽象方法transfer统一了dual stack和 dual queue的操作

```Java
abstract static class Transferer<E> {
    /**
     * 执行put或者take操作
     *
     * @param e 如果非空，值被提供给消费者处理;
     *          如果为空, 返回值给生产者
     * @param timed 操作是否允许超时
     * @param nanos the timeout, in nanoseconds
     * @return 如果非空，表明该返回值将被提供或者接收；如果为空，表明操作犹如超时或者中断而失败
     *         调用者可以通过检查Thread.interrupted来区分这两种情况
     */
    abstract E transfer(E e, boolean timed, long nanos);
}

/** Dual stack */
static final class TransferStack<E> extends Transferer<E> {
    
}
```

#### FixedThreadPool

```java
// 第一种线程池:固定个数的线程池,可以为每个CPU核绑定一定数量的线程数
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(processors * 5);

public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    // 线程池基本大小设为nThreads
    // 线程池最大线程数设为nThreads
    // keepAliveTime设置为0，空闲线程不回收
    // 任务队列采用LinkedBlockingQueue
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```

这种线程池里面的线程被设计成存放固定数量的线程，具体线程数可以考虑为**CPU核数\*N**（N可大可小，取决于并发的线程数，计算机可用的硬件资源等）。可以通过下面的代码来获取当前计算机的CPU的核数。

```Java
int processors = Runtime.getRuntime().availableProcessors();
```

这个实例会复用 固定数量的线程处理一个共享的无边界队列 。任何时间点，最多有 nThreads 个线程会处于活动状态执行任务。如果当所有线程都是活动时，有多的任务被提交过来，那么它会一致在队列中等待直到有线程可用。如果任何线程在执行过程中因为错误而中止，新的线程会替代它的位置来执行后续的任务。所有线程都会一致存于线程池中，直到显式的执行 ExecutorService.shutdown() 关闭。由于阻塞队列使用了LinkedBlockingQueue，是一个无界队列，**因此永远不可能拒绝任务**。LinkedBlockingQueue在入队列和出队列时使用的是不同的Lock，意味着他们之间不存在互斥关系，在多CPU情况下，他们能正在在同一时刻既消费，又生产，真正做到并行。**因此这种线程池不会拒绝任务，而且不会开辟新的线程**，**也不会因为线程的长时间不使用而销毁线程**。这是典型的生产者----消费者问题，这种线程池**适合用在稳定且固定的并发场景**，比如**服务器**。

#### SingleThreadExecutor

```Java
// 单一线程池,永远会维护存在一条线程
ExecutorService singleThreadPool = Executors.newSingleThreadExecutor();
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        // corePoolSize和maximumPoolSize均被置为1
        // keepAliveTime置0，表示线程空闲时不会被回收
        // 任务队列使用无界队列
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

对于newSingleThreadExecutor()来说，也是**当线程运行时抛出异常的时候会有新的线程加入线程池替他完成接下来的任务**。创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，**保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行，所以这个比较适合那些需要按序执行任务的场景。**可以认为是一个单消费者的生产者消费者模式。

#### ScheduledThreadPool

相比于第一个固定个数的线程池强大在 **①可以执行延时任务，②也可以执行带有返回值的任务**

```Java
// 第四种线程池:固定个数的线程池，相比于第一个固定个数的线程池 强大在 ①可以执行延时任务，②也可以执行
// 有返回值的任务。
// scheduledThreadPool.submit(); 执行带有返回值的任务
// scheduledThreadPool.schedule() 用来执行延时任务.
// 固定个数的线程池，可以执行延时任务，也可以执行带有返回值的任务。
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);

FutureTask<String> ft = new FutureTask<>(new Callable<String>() {
    @Override
    public String call() throws Exception {
        System.out.println("hello");
        return Thread.currentThread().getName();
    }
});

scheduledThreadPool.submit(ft);
// 通过FutureTask对象获得返回值.
String result = ft.get();
System.out.println("result : " + result);
// 执行延时任务
scheduledThreadPool.schedule(new Runnable() {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " : bobm!");
    }
}, 3L, TimeUnit.SECONDS);

public static ScheduledExecutorService newScheduledThreadPool(
    int corePoolSize, ThreadFactory threadFactory) {
    return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
}

public ScheduledThreadPoolExecutor(int corePoolSize,
                                       ThreadFactory threadFactory) {
    // 最大线程数设为无限
    // keepAliveTime置0，表明空闲线程不会被回收
    // 缓存队列为DelayedWorkQueue
    super(corePoolSiDelayedWorkQueueze, Integer.MAX_VALUE, 0, NANOSECONDS,
    new DelayedWorkQueue(), threadFactory);
}
```

DelayedWorkQueue是阻塞队列，底层数据结构是RunnableScheduledFuture的数组，初始大小16，可以扩容（grow方法），一次扩容原大小的50%，不能大于MAX_VALUE。