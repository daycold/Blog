# Thread
## Semaphore 信号量
维护当前访问的个数，提供同步机制，控制当前访问的个数。
主要基于 AQS 实现

    与 FixedThreadPool 的差别：
      Semaphore: 多余线程被挂起
      FixedThreadPool: 限制线程数

### acquire 获取许可
### release 释放许可

    fun main() {
        val wrapper = WalkerWrapper()
        for (i in (0..20)) {
            ThreadHelper.POOL.submit { wrapper.walk(Animal(i.toString())) }
        }
        ThreadHelper.POOL.shutdown()
    }
    
    interface Walker {
        fun walk()
    }
    
    class Animal(private val name: String) : Walker {
        override fun walk() {
            println("animal $name walks")
        }
    }
    
    class WalkerWrapper {
        fun walk(walker: Walker) {
            semaphore.acquire()
            println("acquire semaphore")
            Thread.sleep((Math.random() * 10000).toLong())
            walker.walk()
            semaphore.release()
            println("release semaphore")
        }
    
        companion object {
            val semaphore = Semaphore(5)
        }
    }
    结果：
    acquire semaphore
    acquire semaphore
    acquire semaphore
    acquire semaphore
    acquire semaphore
    animal 1 walks
    release semaphore
    acquire semaphore
    animal 0 walks
    release semaphore
    acquire semaphore
    animal 2 walks
    release semaphore
    acquire semaphore
    animal 3 walks
    ......


## AQS (AbstractQueuedSynchronizer)
#### question
    怎么判断当前资源有无持有者？
    一个线程获取资源后怎么阻止其他线程对资源的获取？
    当前持有者释放资源后怎么通知其他线程？
    
### field
state: volatile int, 同步状态

    通过 state 标识当前资源有无持有者

head: volatile Node, 头节点
tail: volatile Node, 尾节点

### Node 
wait queue node class

CLH queue，kind of FIFO queue ?
CLH 锁：通过比较前驱节点的状态来自旋的锁

#### question

1. But being first does not guarantee success; 描述中有句这个， ?
2. 双向列队怎么非公平竞争？

只是在内部类中定义了几个 field。
    
[资源](http://ifeve.com/introduce-abstractqueuedsynchronizer/)

-----------
通过 native 方法比较内存和当前线程的 state 的值，通过 CAS 操作，实现同步（对资源的获取，否则自旋）
提供 hasQueuedPredecessors 方法判断是否有线程等待更久，实现公平竞争（非公平竞争不用管队列）

## Executor
### Executor 接口
仅有一个方法 execute，接受一个 runnable 参数
### ExecutorService 接口
继承 executor 接口，提供对 Callable、Future 和主动关闭执行器的支持
### ThreadPoolExecutor 最终实现类
extends AbstractExecutorService
持有一个 blockingQueue 存放任务
持有一个 Worker 集合
#### Worker 内部类
 extends AQS implementes Runnable
 通过 AQS 实现锁的操作, worker 每次执行任务前加锁，防止被中断（中断需要该线程调用sleep，join，wait，yield 等方法，加锁后该线程不会在其他地方被调用）
 持有一个 final 的 Thread 和一个可变的 Runnable 引用（使用一个线程执行不同的任务）
 如果一个 worker，getTask() == null， 则会销毁
 对于 getTask（）方法，如果设置为允许核心线程过期，获取线程数大于核心线程数，则会调用阻塞列队的 poll 方法并设置超时时间。当前线程数小于核心线程数，则会调用不过期的 take 方法，保持 worker 不销毁
 
 -------
 CachedThreadPool 将核心线程数设置为 0，因而所有 worker 在最大存活期间获取不到任务就会过期
 
 
### ScheduledThreadPoolExecutor
extends ThreadPoolExecutor
#### ScheduledFutureTask 内部类
extends FutureTask implements RunnableScheduledFuture 
相当于 TreadPoolExecutor 的 Worker
拥有 time（下次执行时间），period（执行周期，0: 只一次，正数：time += period, 负数： time = now - period）属性
#### DelayWorkQueue 静态内部类
extends AbstractQueue implements BlockingQueue
指定 ThreadPoolExecutor 的任务列队的实现
放入 ScheduledFutureTask，通过下次执行时间进行排序，将最近执行的放在最前面。

## FolkJoinPool
extends AbstractExecutorService

将一个 task 拆分成多个 task,产出结果是多个 runnable 或者 callable，然后交给线程池执行
task 继承 ForkJoinTask（或者 RecursiveTask 和 RecursiveAction , ForkJoinTask 的子类)
持有一个 WorkQueue 数组
    
### WorkQueue
持有一个 ForkJoinPool 和一个 ForkJoinWorkerTread 的引用，持有一个 task 数组
### ForkJoinWorkerThread
ForkJoinWorkerThread 持有一个 ForkJoinPool 的引用，每次 fork 都会将新的 task 添加到 workQueue (通过传入自身和 ForkJoinPool 注册 workQueue) 中

    question：
            ForkJoinWorkerThread 的 run 方法进行了啥操作？(似乎没有操作？）
            工作窃取算法怎么执行的？
        
## Future
### RunnableFuture
extends Future, Runnable

### FutureTask
 RunnableFuture 的实现

 持有一个 WaitNode 的链表（任务中可能嵌套多线程任务）
 当某个节点抛出异常时，抛出异常
 
 持有一个 outcome
 run 方法成功返回会 set outcome
 调用 get 的时候，如果 waitNode 有节点没有执行完，调用 yield 阻塞当前线程
 通过 get 方法获取结果
#### WaitNode
持有一个当前线程的引用 thread 和一个 WaitNode 的引用 next

## object, thread
wait: 让出 cpu 和锁（需要 notify 唤醒）
sleep： 仅让出 cpu
notify/notifyAll: 唤醒竞争该对象失败而进入 wait 状态的该对象所在的线程
yield：退出 running，进入 runnable 状态，仅让出 cpu
join: 一个 synchronized 方法，轮询存活状态并 wait

         while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
        }
        join 方法先被 synchronized 修饰，如果无法获取到该线程的锁则会阻塞。获取到后会查看状态，如果未完成，调用 thread.join 方法的线程释放锁，等待 thread 执行完成将其唤醒。
        yield 和 sleep 都是 Thread 的静态方法，锁是落在对象上的，因而这两个方法不存在操作锁的方式
        对于从wait中被notify的进程来说，它在被notify之后还需要重新检查是否符合执行条件，如果不符合，就必须再次被wait，如果符合才能往下执行。所以：wait方法应该使用循环模式来调用
        thread 在结束是会调用一次 notifyAll，唤醒持有该线程对象所有 wait 的线程

## Condition
AQS 的内部类 ConditionObject 实现了 Condition 接口

### ConditionObject
持有一个双端队列的头尾节点引用，当前线程调用 await 方法后构造成节点加入队尾。
一个锁上可以有多个 Condition，持有多个队列
AQS 自身也持有双端列队，所以 condition 相当于对等待队列作分组?
AQS 本身仅实现了线程间的竞争和等待队列，condition 实现了占有锁后让出锁的实现


## Atomic 类
通过 unsafe.compareAndSet 实现线程安全

例：

    多线程正数自增：
    private final AtomicInteger ai = new AtomicInteger(0);
    
    public void increase() {
        int i = ai.get();
        while(!ai.compareAndSet(i, i + 1)) {
            i ++;
        }
    }
    
    一段代码在多线程下只运行一次
    private final AtomicBoolean ab = new AtomicInteger(true);
    
    public void done() {
        if (ab.compareAndSet(true, false)) {
            // doSth
            ab.compareAndSet(false, true);
        }
    }
    


