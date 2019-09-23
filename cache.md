# CACHE
## CacheManager
将 cacheManager 注册成 bean 后，@Cacheable 将通过 cacheManager 获取缓存容器

### RedisCacheManager
通过 redis 的 keys 命令通过正则获取所有匹配的 key，再 del，实现 clear

