# COROUTINES
## 基础操作

function | details
--- | ---
GlobalScope | 后台启动一个全局协程（非阻塞）launch 返回 job, async 返回 deferred
delay | 挂起协程（不阻塞线程）
runBlocking | 阻塞主线程直到里面代码运行结束
Job.join | 等待job（无结果）协程结果，阻塞主线程
coroutineScope | 申明协程作用域并不阻塞当前线程
Channel | 管道（类似阻塞列队）
Deferred | is Job, 有结果
async | 启动单独一个协程，返回 Deferred
launch | 启动协程，返回 Job
yield | 挂起协程，分发到 dispatcher 队列,相当于 Thread.yield

其他表达式

function | details
--- | ---
select | 同时等待多个挂起函数，并选择第一个可用的(其余的会继续运行完成）

有挂起操作或者被 suspend 标记的方法必须被 suspend 申明
launch 等方法会在一开始给没有调度器的 CoroutineContext 分配默认调度器

### select

     val job = CoroutineScope(context).async {
          select<String> {
              async {
                  val time = System.currentTimeMillis()
                  delay(2000)
                  println(System.currentTimeMillis() - time)
              }.onAwait.invoke { "job 1" }
              async {
                  val time = System.currentTimeMillis()
                  delay(1000)
                  println(System.currentTimeMillis() - time)
              }.onAwait.invoke { "job 2" }
          }
      }
      println(job.await())
      println(System.currentTimeMillis() - mills)
    
返回结果为：

    1023
    2006
    job 2
    2078

## Mutex
互斥锁
Mutex().withLock {...} 等价于 val mutex = Mutex();try {mutex.lock;...}finally{mutex.unlock}

## ThreadContextElement
协程中的 ThreadLocal

## CoroutineDispatcher
协程调度器(也是继承自 CoroutineContext)
抽象类，继承自 AbstractCoroutineContextElement，ContinuationInterceptor
在调用 CoroutineScope.newCoroutineContext 方法时会给 CoroutineContext 分配调度器(CoroutineContext 没有 ContinuationInterceptor 的 key 时)

     class TestCoroutineDispatcher(override val executor: ThreadPoolExecutor) 
     : ExecutorCoroutineDispatcher() {
        override fun close() {
            executor.shutdown()
        }

        override fun dispatch(context: CoroutineContext, block: Runnable) {
            executor.submit {
                block.run()
            }
        }
    }
    
    fun main()  {
        val threadPool = Executors.newFixedThreadPool(5)
        val scope = CoroutineScope(TestCoroutineDispatcher(threadPool))
        scope.async {
            println(Thread.currentThread().name)
        }
        println(Thread.currentThread().name)
        Thread.sleep(200)
    }
    
    输出：
        main
        pool-1-thread-1
    这样携程完全交给线程池调度，相当于一个线程池


### isDispatchNeeded
实验性功能
如果会调度到别的线程上，则返回 true
### dispatch
将任务调度到 CoroutineContext 指定的线程上
### dispatchYield
实验性功能
任务调度到 CoroutineContext 指定线程上，当前的 dispatch 会由 yield 触发

## AbstractCoroutineContextElement
抽象类，继承 Element
添加构造器，实现抽象变量 key

## ContinuationInterceptor
继承 Element
### interceptContinuation
传入 Continuation，返回 Continuation，对传入参数进行包装
### releaseInterceptedContinuation
仅当 interceptContinuation 的参数和返回不是同一个对象时会调用该方法
释放老的？
## Continuation
持有一个 CoroutineContext 的引用
每次挂起时做参数（隐式）传入，封装协程恢复后的执行逻辑

suspendCoroutine，createCoroutine, startCoroutine，扩展 suspend 修饰对象（方法）的三个方法，传入 Continuation 做参数
挂起时会使用expect标记的方法创建一个新的Continuation，将执行过程分为多个Continuation片段，由状态机(可能是SequenceBuilderIterator?)保证各个状态的执行顺序
### resumeWith
接受 Result 参数，处理结果
## Result
持有 isSuccess 和 isFailure 两个引用
## CoroutineContext
协程上下文
一个类似 map 的数据结构，类似 Map<Key<T>, T>, key中会指定value的类型

接口仅申明如下方法
fold, get, minusKey，plus
### fold
遍历所有 Element 并接受 lambda 操作
### get
通过 key 获取泛型对应的 Element
### minusKey
返回一个不包含参数 key 的 coroutineContext
### plus
合并两个 coroutineContext

## Element
继承自 CoroutineContext
申明抽象 val key: Key<*>
对 CoroutineContext 中的三个抽象方法使用 key 提供默认实现

## Key
空的泛型接口，申明泛型 E: Element

## CoroutineScope
持有一个抽象的 CoroutineContext 引用
携程作用域

当协程在 CoroutineScope 中启动时，将继承父协程的上下文，它的 job 将成为父协程的子 job。父协程取消时随之取消
父协程会等待所有子协程执行完再结束，不需要显式 join

使用 runBlocking 创建协程时，如果不指定 coroutineContext，则会创建一个 EventLoop（继承调度器）
createEventLoop() 被 expect 关键字标记，没找到 dispatch 的方法实现

## Job
继承自 Element
### start
开始协程
### cancel
取消协程
### join
阻塞当前协程直到 job 完成
### onJoin
针对 select 表达式
### children
获取所有子 job
### attachChild
添加子 job
### invokeOnCompletion
完成时回调
## JobSupport
继承自 Job, ChildJob, ParentJob, SelectClause0 (select从句)
实现所有接口
## AbstractCoroutine
抽象类
继承自 JobSupport， Job， Continuation，CoroutineScope

申明了 onStart, onCompleted, onCancelled, onCompletionInternal, onStartInternal 等钩子方法

### initParentJob
使用父 job 的上下文初始化父 job
### delay
将job生成task放入延迟队列中,让出cpu。
    val coroutineDispather = newFixedThreadPoolContext(2, "threadPoolContext")
    CoroutineScope(coroutineDispather).launch {
        repeat(5) {
            println(Thread.currentThread().name)
            delay(200)
        }
    }
    println(Thread.currentThread().name)
    Thread.sleep(1500)
    结果:
    main
    threadPoolContext-1
    threadPoolContext-2
    threadPoolContext-2
    threadPoolContext-1
    threadPoolContext-1

    runBlocking {
        launch {
            repeat(5) {
                println(Thread.currentThread().name)
                delay(200)
            }
        }
        println(Thread.currentThread().name)
        Thread.sleep(1000)
    }
    结果:
    main @coroutine#1
    main @coroutine#2
    main @coroutine#2
    main @coroutine#2
    main @coroutine#2
    main @coroutine#2

开启默认调度器的协程默认会新建BlockingEventLoop对象调度，该调度器只有一个线程。
因而，runBlocking里面是一个线程，lauch里面是一个新的线程。

## 总结
协程作用域中申明创建调用协程。
协程作用域持有协程上下文的信息,用以在挂起时保存协程的状态。
协程作用域相当于一个 dsl 作用域，通过 dsl 来创建协程，通过协程上下文来传递信息。
协程可以配合nio进行调度,资源未准备好时挂起，而不阻塞线程进行其他操作。
将竞争公共资源的部分放在协程中时，协程可以在竞争失败时挂起，而不阻塞线程进行其他操作。
所以协程的优点是可以调度线程内代码的执行顺序，减少线程阻塞产生的资源浪费。
修改dispatcher的调度方式（如使用线程池调度），等同于多线程
协程的本质是使用一个线程（线程池）和任务队列调度任务


