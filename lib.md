# Lib
## Random
构造函数可以指定seed，否则通过静态变量和当前纳秒计算得出，并更新该静态量。
在更新静态常量和计算随机值时，会有
do {
    a = atomicLong.get()
    b = xxxL
} while(!atomicLong.compareAndSet(a, b)
该循环仅解决多线程下的一致性问题。单线程只会执行一次。

