# CACHE
## CacheManager
将 cacheManager 注册成 bean 后，@Cacheable 将通过 cacheManager 获取缓存容器

### RedisCacheManager
通过 redis 的 keys 命令通过正则获取所有匹配的 key，再 del，实现 clear

## caffeine 过期时间探究

getIfPresent -> scheduleDrainBuffers -> executor().execute PerformCleanupTask

在 PerformCleanupTask 对象中，调用链为
run -> performCleanUp -> maintenance -> expireEntries

expireAfterAccess 过期策略
cache 持有一些 accessOrderDeque，通过当前时间缓存时间和过期时间的对比执行过期策略。

expireAfterWrite:
只有 writeOrderDeque

expireVariableEntries:
通过 TimerWheel.advance 执行过期
TimerWheel 作用？

总体来说，在执行 cache.get 类方法时，倘若拿到一个过期数据，触发缓存过期的异步执行

### AccessOrderDeque
继承自 AbstractLinkedDeque
限制泛型为 AccessOrder 的子类。
AccessOrder 持有前驱节点和后驱节点的 setter 和 getter

### TimerWheel
持有一个 cache
持有一个 node 的二维数组

