# Redis 缓存淘汰策略

## Redis 内存淘汰策略

* `noeviction` 不删除策略
* `allkeys-lru` 所有key通用,优先删除最近最少使用(LRU)的key
* `allkeys-random` 所有key通用,随机删除一部分key
* `volatile-lru` 只限于设置expire的部分,优先删除最近最少使用(LRU)的key
* `volatile-random` 只限于设置了expire的部分,随机删除一部分key
* `volatile-ttle` 只限于设置了expire的部分,优先删除剩余时间(ttl time to live)短的key

## 常见的缓存失效策略

* `FIFO` first in first out 先进先出
* `LFU` less frequently used 一直以来最少使用的元素会被清理掉
* `LRU` less recently used 最近最少使用



## 缓存更新策略

* `cache aside`
* `read through`
* `write through`
* `write behind caching`