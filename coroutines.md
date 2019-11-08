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
### resumeWith
接受 Result 参数，处理结果
## Result
持有 isSuccess 和 isFailure 两个引用
## CoroutineContext
协程上下文

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

当协程在 CoroutineScope 中启动时，将继承父协程的上下文，它的 job 将成为父协程的子 job。父协程取消时随之取消
父协程会等待所有子协程执行完再结束，不需要显式 join

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


