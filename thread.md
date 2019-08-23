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
## Executor
## FolkJoinPool


