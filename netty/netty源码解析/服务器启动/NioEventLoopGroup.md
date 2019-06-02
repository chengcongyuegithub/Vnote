# NioEventLoopGroup
```
 EventLoopGroup bossGroup= new NioEventLoopGroup();
 EventLoopGroup workerGroup=new NioEventLoopGroup();
```
## EventLoopGroup

```
Special {@link EventExecutorGroup} which allows registering {@link Channel}s that get
 * processed for later selection during the event loop.
```
* 从上面的信息我们可以知道EventLoopGroup是特殊的EventExecutorGroup,**也就是EventLoopGroup是继承EventExecutorGroup的**
* 在事件循环期间,这个EventLoopGroup它还允许Channel注册到选择器上去
大概意思就是EventLoopGroup里面有多个EventLoop,这个EventLoop是真正做事情的,它做的事情就是完成选择器的注册
```
 /**
           * Return the next {@link EventLoop} to use
     */
    @Override
    EventLoop next();

    /**
     * Register a {@link Channel} with this {@link EventLoop}. The returned {@link ChannelFuture}
     * will get notified once the registration was complete.
     */
    ChannelFuture register(Channel channel);

    /**
     * Register a {@link Channel} with this {@link EventLoop} using a {@link ChannelFuture}. The passed
     * {@link ChannelFuture} will get notified once the registration was complete and also will get returned.
     */
    ChannelFuture register(ChannelPromise promise);
```
* 其中next相当于是获取到下一个**真正干活的**
* ChannelFuture register(Channel channel)和ChannelFuture register(ChannelPromise promise)的作用基本是相同的,就是完成注册的,这个操作是**异步**的,
因为是Future的子类,ChannelPromise其中包括Channel,完成的任务就是注册通道

**总结一下**
EventLoopGroup里面有多个EventLoop,这个类的作用就是在循环中将来的连接注册到选择器上去.


## NioEventLoopGroup
###  NioEventLoopGroup线程数的确定
```
/**
 * {@link MultithreadEventLoopGroup} implementations which is used for NIO {@link Selector} based {@link Channel}s.
 */
```
基于NIO的多线程MultithreadEventLoopGroup
然后我们看一下构造方法,这个比较多,我们直接看最后的
```
 public NioEventLoopGroup(int nThreads, Executor executor, final SelectorProvider selectorProvider,
                             final SelectStrategyFactory selectStrategyFactory) {
        super(nThreads, executor, selectorProvider, selectStrategyFactory, RejectedExecutionHandlers.reject());
    }
```
也就是使用的MultithreadEventLoopGroup中的构造方法
```
protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
        super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
    }
```
```
private static final int DEFAULT_EVENT_LOOP_THREADS;

    static {
        DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
                "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));

        if (logger.isDebugEnabled()) {
            logger.debug("-Dio.netty.eventLoopThreads: {}", DEFAULT_EVENT_LOOP_THREADS);
        }
    }
```
上面的代码就完成了线程数的确定
```
chengcongyue@chengcongyue:~/下载$ cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
1  //当前电脑的CPU的个数
chengcongyue@chengcongyue:~/下载$ cat /proc/cpuinfo| grep "cpu cores"| uniq
cpu cores	: 2   //CPU中的核的个数
chengcongyue@chengcongyue:~/下载$ cat /proc/cpuinfo| grep "processor"| wc -l
4                 //CPU逻辑核的个数
```
DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
                "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));这一句得到的就是当前CPU逻辑核的个数,对它*2就是NioEventLoopGroup的线程数量.
### NioEventLoopGroup中线程的创建
我们在知道了线程数量的创建之后,我们就要进入到这个构造方法中了
```
protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
        super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
    }
```
我们进入到其中
在其中完成了线程的创建
```
children[i] = newChild(executor, args);
```
其中的executor,可以是传进来的,但是在null的时候,我们会自己创建
```
 if (executor == null) {
            executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
        }
```
以上的代码就是在创建线程之前执行的.**newDefaultThreadFactory**实现的就是在创建线程的时候,同时可以赋属性,同时也实现了**创建线程,和开启线程的解耦**

## Executor
### part1
```
An object that executes submitted {@link Runnable} tasks. This
 interface provides a way of decoupling task submission from the
 mechanics of how each task will be run, including details of thread
 use, scheduling, etc.  An {@code Executor} is normally used
 instead of explicitly creating threads. For example, rather than
 invoking {@code new Thread(new(RunnableTask())).start()} for each
 of a s
 et of tasks, you might use:
  
```
一个类用来执行提交的任务的.
```
Executor executor = <em>anExecutor</em>;
  executor.execute(new RunnableTask1());
  executor.execute(new RunnableTask2());
```
这个接口通过了一种方式,**将任务的提交和任务的运行分离**
```
new Thread(new(RunnableTask())).start()其中任务的提交和任务的运行就合在了一起
```
### part2
```
 * However, the {@code Executor} interface does not strictly
 * require that execution be asynchronous. In the simplest case, an
 * executor can run the submitted task immediately in the caller's
 * thread:
 *
 *  <pre> {@code
 * class DirectExecutor implements Executor {
 *   public void execute(Runnable r) {
 *     r.run();
 *   }
 * }}</pre>
```
在上面的方式中,就只有一个线程,这个线程就是DirectExecutor
上面doc的意思是,Executor不一定是异步的,在最简单的场景中,一个Executor可以同步的跑一个线程
比较常见的情况如下
```
 * More typically, tasks are executed in some thread other
 * than the caller's thread.  The executor below spawns a new thread
 * for each task.
 *
 *  <pre> {@code
 * class ThreadPerTaskExecutor implements Executor {
 *   public void execute(Runnable r) {
 *     new Thread(r).start();
 *   }
 * }}</pre>
```
这样的话就是异步执行,其中的也就是**run和start的区别**
### part3
```
* Many {@code Executor} implementations impose some sort of
 * limitation on how and when tasks are scheduled.  The executor below
 * serializes the submission of tasks to a second executor,
 * illustrating a composite executor.
 *
 *  <pre> {@code
 * class SerialExecutor implements Executor {
 *   final Queue<Runnable> tasks = new ArrayDeque<Runnable>();
 *   final Executor executor;
 *   Runnable active;
 *
 *   SerialExecutor(Executor executor) {
 *     this.executor = executor;
 *   }
 *
 *   public synchronized void execute(final Runnable r) {
 *     tasks.offer(new Runnable() {
 *       public void run() {
 *         try {
 *           r.run();
 *         } finally {
 *           scheduleNext();
 *         }
 *       }
 *     });
 *     if (active == null) {
 *       scheduleNext();
 *     }
 *   }
 *
 *   protected synchronized void scheduleNext() {
 *     if ((active = tasks.poll()) != null) {
 *       executor.execute(active);
 *     }
 *   }
 * }}</pre>
 *
```
以上的情况就是将线程传给第二个执行者
### part4
```
 * The {@code Executor} implementations provided in this package
 * implement {@link ExecutorService}, which is a more extensive
 * interface.  The {@link ThreadPoolExecutor} class provides an
 * extensible thread pool implementation. The {@link Executors} class
 * provides convenient factory methods for these Executors.
```
ExecutorService继承了Executor.
ThreadPoolExecutor通过了可拓展的线程池
Executors提供了工厂方法,其功能就是创建Executor
### part5
```
 * <p>Memory consistency effects: Actions in a thread prior to
 * submitting a {@code Runnable} object to an {@code Executor}
 * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 * its execution begins, perhaps in another thread.
```
内存一致性,当前线程在提交任务(runnable接口的内容)给Executor之前,它的初始化可能在另一个线程中就完成了