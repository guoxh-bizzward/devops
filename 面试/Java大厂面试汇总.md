# Java大厂面试10个知识点汇总

https://segmentfault.com/a/1190000039153631

## Java基础

* Object 类的所有方法

* 八种基本数据类型&取值范围

* 自动拆装箱 & 包装类的缓存机制
* ThreadLocal String&不可变性 StringBuffer StringBuilder
* ArrayList LinkedList HashMap(重点，可以花一天时间彻底掌握) LinkedHashMap TreeMap HashSet LinkedHashSet TreeSet源码
* 异常体系
* Java8 理解函数式编程 流式操作  CompletableFuture

## 并发编程

* 并发和并行
* 线程池的原理， 理解每个参数的意义
* Thread Runable Callable
* AQS 常常配合synchronized一起 Condition
* Lock ReentrantLock ReentrantReadWriteLock
* Semaphore CountDownLatch CyclicBarrier
* Atmoic
* ConcurrentHashMap BlockingQueue CopyOnWriteArrayList
* Executors
* 线程的状态转换  wait sleep notify notifyAll join park
* 停止线程的方法
* 线程模型 BIO NIO AIO，理解Netty
* 并发包的那些东西

## JVM

《深入理解Java虚拟机》

* JVM内存结构
* 常用调优参数
* GC算法
* gc的工具：CMS G1 ZGC
* Java内存模型
  * 可见性 原子性 顺序性,happen-before,内存屏障，volatile,final
  * synchronized（对象头） 偏向锁 轻量级锁 重量级锁 monitor 锁优化 锁消除 锁粗化 自旋锁 可重入锁
* 常用工具以及命令
* 类加载过程
* 对象创建过程
* fullGC排查思路
* 工具 jps，jstack，jmap，jstat，jconsole，jinfo，jhat，javap，Arthas

## 设计模式

* 创建型
* 结构型
* 行为型

##　MYSQL

* 常用sql以及内置函数
* 索引
  * hash索引
  * B+ Tree
  * 聚集索引 非聚集索引
  * 回表
  * 联合索引 最左前缀&覆盖索引&索引下沉
* 事务&隔离级别&实现原理
* 锁的使用&锁算法&行锁&表锁&乐观锁&悲观锁
* 连接查询的原理（算法）
* binlog&使用场景
* 分库分表：垂直&水平
* 工具 explain

## Redis

《redis的设计与实现》

* redis五种类型&适用场景 字符串 列表 哈希 集合 有序集合
* 了解底层数据结构 SDS 链表 字典 跳跃表 整数集合 压缩列表
* 淘汰策略
* RDB & AOF
* 复制功能的实现
* Sentinel & 集群的结构
* 发布订阅 & 事务
* 分布式锁
* 常见问题&解决方案 缓存穿透 缓存击穿 缓存雪崩

##　Web

* 三次握手　四次挥手
* http1.0 http1.1 http1.2
* http get VS post
* https 加密
* forward redirect
* CRSF XSS攻击与防范
* 常见的web请求返回的状态码
* 