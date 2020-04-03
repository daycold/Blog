# Thread
cpu每次只运行线程的一个时间片。在存在竞争的过程中，多个线程的时间片可能交替执行。加锁后，没获得锁的线程不会参与竞争。


## AQS (AbstractQueuedSynchronizer)
#### question
    怎么判断当前资源有无持有者？
    一个线程获取资源后怎么阻止其他线程对资源的获取？
    -- 通过 CAS 进行同步操作
    当前持有者释放资源后怎么通知其他线程？
    -- 所有线程都不会休眠，进入一个等待队列。队列的头部会不停进行 CAS 操作，当成攻后就获得锁并出队列。根据状态判断获取锁失败的线程自旋还是休眠。
### field
state: volatile int, 同步状态

    通过 state 标识当前资源有无持有者

head: volatile Node, 头节点
tail: volatile Node, 尾节点

### Node 
wait queue node class

CLH queue，kind of FIFO queue ?
CLH 锁：通过比较前驱节点的状态来自旋的锁

#### doAcquireShard
创建一个 Node,添加到等待队列中
当前驱节点为头节点（头结点拥有锁）时，尝试获取锁（如果头结点释放了锁则可用获取到锁，并且将自身置为头结点）
判断获取失败是否阻塞线程，如果需要阻塞则调用 LockSupport.park 阻塞线程（在 release 锁的时候会调用 LockSupport.unpark 释放队列的第一个阻塞线程）。
#### question

1. But being first does not guarantee success; 描述中有句这个， ?可能因为跟其他线程一起竞争cpu导致失败
2. 双向列队怎么非公平竞争？

只是在内部类中定义了几个 field。
    
[资源](http://ifeve.com/introduce-abstractqueuedsynchronizer/)

-----------
通过 native 方法比较内存和当前线程的 state 的值，通过 CAS 操作，实现同步（对资源的获取，否则自旋）
提供 hasQueuedPredecessors 方法判断是否有线程等待更久，实现公平竞争（非公平竞争不用管队列）

## Condition
AQS 的内部类 ConditionObject 实现了 Condition 接口

### ConditionObject
持有一个双端队列的头尾节点引用，当前线程调用 await 方法后构造成节点加入队尾。
一个锁上可以有多个 Condition，持有多个队列
AQS 自身也持有双端列队，所以 condition 相当于对等待队列作分组?
AQS 本身仅实现了线程间的竞争和等待队列，condition 实现了占有锁后让出锁的实现

## Semaphore 信号量
维护当前访问的个数，提供同步机制，控制当前访问的个数。
主要基于 AQS 实现

     for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
                return remaining;
    }


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

## Phaser
允许并发多阶段任务 [文档](https://segmentfault.com/a/1190000015979879)
在每一步结束的位置对线程同步，所有线程多完成这一步才允许执行下一步
与 CountDownLatch，CyclicBarrier 相似的功能，更为动态？
多阶段栅栏，可以在初始时设定参与线程数，也可以中途注册/注销参与者，当到达的参与者数量满足栅栏设定的数量后，会进行阶段升级（advance）
不依赖 AQS

### register
新注册一个 party
### arrive
抵达 Phaser，到达线程计数器加一，并不等待其他未完成线程
### arriveAndDeregister
抵达 Phaser，到达线程计数器加一，并不等待其他未完成线程，注销此线程注册的 party
### arriveAndAwaitAdvance 
抵达 Phaser，到达线程计数器加一，等待其他未完成线程
### awaitAdvance
在指定阶段 phase 等待其他线程到达屏障点

### internalAwaitAdvance(phase, node)
调用 releaseWaiters(phase -1）唤醒所有阻塞的当前阶段任务
将node（或者创建新的节点）添加到下一阶段队列中，并阻塞
所有线程已达时，DNN用 releaseWaiters(phase)唤醒所有阻塞的当前阶段的任务

### releaseWaiters(phase)
Phaser持有两个 node 队列，根据 phase 的奇偶交替扮演当前阶段和下一阶段任务队列。

## CountDownLatch
使用 AQS 共享锁实现同步并发任务。
state 无法重置（只能使用一次）。
重写的 tryAcquireShared，返回结果与参数无关
倒数计数器，初始时设定计数器值。
通过比较 getCount 的值和 await 方法的调用，可以实现按次序执行

    thread1 {
        while (true) {
            if (getCount() == 2) {
                doSth...
            } else {
                wait()
            }
        }
    }
    
    thread2 {
        while(true) {
            if (getCount() == 1) {
                doSth...
            } else {
                wait()
            }
        }
    }
    
    能保证先执行 thread1 再执行 thread2

### await
休眠线程，唤醒一个休眠线程
aqs.acquireSharedInterruptibly(1)，当当前状态不为 0，则会阻塞线程（与参数 1 无关）

### countDown
释放共享锁，计数器-1，唤醒一个休眠线程

### getCount
未释放的锁的数量（当前计数器的值）
aqs.releaseShared(1)

## CyclicBarrier
持有一个 ReentrantLock, 一个 Condition，持有一个 barrierCommand(Runnable),所有线程完成任务后触发。
循环栅栏，初始时设定参与线程数，当线程到达栅栏后，会等待其它线程的到达，当到达栅栏的总数满足指定数后，所有等待的线程继续执行
使用 ReentrantLock 和 Condition 做同步控制

### reset
唤醒所有休眠线程，重置计数器
### await
未完成线程计数器减一。
如果计数器为 0，调用 barrierCommand，并唤醒所有线程，重置计数器，直接返回。
否则，调用 Condition.await 休眠线程

## Exchanger
类似于 CyclicBarrier
Thread1线程到达栅栏后，会首先观察有没其它线程已经到达栅栏，如果没有就会等待，如果已经有其它线程（Thread2）已经到达了，就会以成对的方式交换各自携带的信息
[参考](https://segmentfault.com/a/1190000015963932)

持有一个 participant，继承 ThreadLocal<Node>,持有一个 Node 数组 arena 和 Node slot, int bound

    大致思路：
        每个线程持有各自的 node。
        使用各自的 node 通过 cas 对 slot 赋值。
        如果原来 slot 不为空，则对其置空值，得到slot的值
        修改原来的 slot，传入该线程 node 中的值，并唤醒slot指向的线程
        如果原来的slot 为空，则赋值并阻塞当前线程
### exchange

#### 实践

    val exchanger: Exchanger<Message> = Exchanger()
    val thread1 = thread {
        sleep(200)
        val message = Message()
        message.message = "success"
        println("thread1: ${exchanger.exchange(message).message}")
    }
    val thread2 = thread {
        val message = Message()
        val newMessage = exchanger.exchange(message)            // -----------1
        message.message = "failure"                             // -----------2
        println("thread2: ${newMessage.message}") 
    }
     输出 thread1: ; thread2: success
     将 1，2 两行互换后，输出 thread1: failure ; thread2: success
     当添加第三个 thread 时，第三个 thread 将一直处于阻塞情况。
     
调用 exchange 方法后不能再在本线程修改对象来改变其他线程的获取到的结果

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
 通过 AQS 实现锁的操作, worker 每次执行任务前加锁，防止任务被中断（各种原因引起的阻塞，如sleep，join，wait，yield 等方法，加锁后该线程不会在其他地方被调用）
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
        
## object, thread
竞争锁失败的线程依旧是活跃的线程，就绪状态。
调用 wait 的线程被阻塞，需要 notify 才能进入就绪状态。

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


## NIO
非阻塞型 IO，数据未准备好时，不进行等待，返回异常，此时可由用户控制继续访问或者先做别的事
### Selector
使用独立线程轮询注册的信道，满足条件时通知工作线程

## BlockingQueue

基础方法

method | detail
--- |---
add | 添加元素，列队满了抛出 IllegalStateException
offer | 添加元素，列队满了（或等待一定时间后依旧满了）返回 false
put | 添加元素，列队满了阻塞
take| 获取元素，列队为空时阻塞
poll | 获取元素，列队为空（或等待一定时间后仍然为空）返回 null

### ArrayBlockingQueue
持有一个 ReentrantLock 引用，
两个 Condition 引用 notEmpty, notNull

#### offer(E)
简单的 lock

        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }

#### offer(E, long, TimeUnit)
队列满时将线程加到 notFull 的队列下指定时间

        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                if (nanos <= 0)
                    return false;
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }


#### put(E)
列队满是阻塞

        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }

     
## 伪缓存

缓存系统（如 cpu 缓存）中以缓存行的形式进行存储。cpu 与主内存之间有几层缓存。越靠近 cpu 访问速度越快。
缓存行 64 字节。
两个数据存储在同一个缓存行，被两个线程修改时，处理器之间会发起通信，使该缓存行只被一个处理器操作。完成后发起通信以通知其他处理器。该过程会浪费资源。

### 缓存行填充
Java 程序的对象头固定占 8 字节(32位系统)或 12 字节( 64 位系统默认开启压缩, 不开压缩为 16 字节)，所以我们只需要填 6 个无用的长整型补上6*8=48字节，让不同的 VolatileLong 对象处于不同的缓存行，就避免了伪共享

@sun.misc.Contended 注解会实现该操作


## 锁
无锁、偏向锁，轻量级锁，重量级锁
### 偏向锁
偏向锁有四个状态：未占有 NEUTRAL， 偏向 BIASED，撤销 REVOKING，已退化 DEFAULT
减少使用cas操作，默认锁只会由单一线程获得。
第一种场景是当锁还没有被任何线程获取过处于初始状态NEUTRAL时。在这种状态下，想要获取锁的线程首先会测试锁的状态，然后会使用CAS操作将锁状态中当前持有者线程置为自己的线程ID，当一个线程成功的在锁状态中写入自己的线程ID时，即成为该偏向锁的持有者，锁的状态会由NEUTRAL变为BIASED，同时该锁也会被这个线程获取。

第二种场景是偏向锁的当前持有者线程重新获取锁。在这种场景下，持有者线程只需要检测所的状态位保存自己的线程ID，再测试该锁状态不为REVOKING，最后对布尔状态位进行赋值即可。如果该锁被撤销了，锁的当前持有线程的获取动作会失败，需要进入获取默认锁的路径。

第三种场景发生在第一个非偏向锁持有者尝试获取同一个锁时，通过测试锁当前状态为BIASED，并且锁状态中保存的线程ID不是自己的线程ID可以知道进入此场景。该线程会通过CAS操作将锁的状态由BIASED改为REVOKING。因为偏向锁的当前持有线程可能会随时改变锁的状态(比如随时获取、释放偏向锁)，所以上面的CAS操作会使用循环进行多次尝试，且每次尝试CAS之前都必须重新读取一下锁状态。需要注意的是，可能会存在多个非持有者线程同时使用CAS尝试将锁状态从BIASED改为REVOKING，CAS则保证最终只有一个线程修改成功，修改成功的线程会变成该锁的撤销者(revoker)。其他修改失败的线程会被通知该锁现在已经不是偏向状态了，从而进入自旋状态直到撤销者成功的完成偏向锁的撤销过程，将锁状态改为DEFAULT，这些线程最终会转向去获取普通锁。偏向锁的撤销者会等待偏向锁的当前持有者释放锁，然后撤销者线程会成功的将锁变成普通锁，然后也会成为该普通锁的第一个获取者。

第四种场景发生在锁状态为DEFAULT时，进入这个场景有三步：首先，该线程尝试获取初始状态(NEUTRAL)或者偏向锁(BIASED)失败；然后，该线程通过获取普通锁的路径成功获取普通锁；最后，该线程成功获取锁，并从等待锁的阻塞中被唤醒。

### 自旋锁
线程的阻塞和唤醒需要CPU从用户态转为核心态，频繁的阻塞和唤醒对CPU来说是一件负担很重的工作。
在循环中尝试获取锁，不阻塞不做其他工作。需要限定循环次数。
当对锁占用时间很短时，减轻cpu的开销。
### 轻量级锁
使用cas修改锁对象头的指针，失败则自选。自旋锁仍失败时，膨胀为重量级锁
### 重量级锁
包含一个竞争锁的队列和阻塞队列


