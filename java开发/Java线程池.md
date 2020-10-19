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

### 四种线程池

#### newCacheThreadPool

```Java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

// 构造器
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

返回ThreadPoolExecutor实例，corePoolSize为0，maximumPoolSize为Integer.MAX_Value，keepAliveTime为60L，workQueue为SynchronousQueues（同步队列）。

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
    // ThreadGroup和Thread的关系相当于元素和集合的关系，ThreadGroup之间的关系是树的关系。main方法执行后，将自动创建system线程组和main线程组，main方法所在线程存放在main线程组中。
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

