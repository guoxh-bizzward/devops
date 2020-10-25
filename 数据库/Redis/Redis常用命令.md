# Redis 常用命令

## Redis 基本数据类型

### string(字符串)

string类型是二进制安全的,可以包含任意数据,包括jpg图片或者序列化的对象.

string类型的值最大能存储512MB

### hash(哈希)

hash是一个键值对的集合,每个hash可以存储2^32-1 个键值对(40多亿)

hash是一个string类型的field和value的映射表.hash特别适合存储对象.

### list(列表)

一个列表最多可以存储2^32-1 个元素

列表是一个简单的字符串列表,按照插入顺序排序.可以添加一个元素到列表的头部(左边)或者尾部(右边)

### set(集合)

set是一个string类型的无序集合.集合中最大的成员数为 2^32 - 1 

集合是通过哈希表实现的,所以添加,删除,查找的复杂度都是O(1)

### zset(有序集合)

zset和set一样也是string元素的集合,且不允许有重复成员

不同的是每个元素都会关联一个double类型的分数,redis正是通过分数为集合中的成员进行从小到大的排序

zset的成员是唯一的,但分数(score)可以重复.

集合是通过hash表实现的,所以添加,查找,删除的复杂度都是O(1)

## 常用命令

 ### redis 连接命令

> redis-cli -h host -p port -a password

```
redis-cli -h localhost -p 6379 -a 123456
```

### keys命令

基本语法

> COMMAND KEY_NAME

* 查找符合条件的key

> keys pattern

* 删除key 

> del key

* 检查key是否存在

> exists key

* 返回key所存储的值类型

> type key

* 为key设置过期时间

> expire key seconds #以秒计
>
> expireat key timestamp # 接受的时间参数是Unix时间戳(uninx timestamp)
>
> pexpire key milliseconds #以毫秒计
>
> pexpireat key millseconds-timestamp #设置key过期时间的时间戳以毫米计

* 移除key的过期时间

> persist key

* 查看key 剩余过期时间

> pttl key # 以毫秒为单位返回key的过期时间
>
> ttl key  # 以秒为单位返回key的过期时间

* 重命名key

> rename key newKey
>
> renamenx key newKey # 仅当redis中newkey不存在时,将key改名为newKey

* 序列化给定key

> dump key

### string 命令

* 赋值

> set key value

* 取值

> get key

* 返回key对应value的字符串子集(相当于substring(start,end))

> getrange key start end

* 将给定的key的值设置为value,并返回该key对应的oldvalue

> getset key value

* 获取一个或者多个key的值

> mget key1 [key2...]

* 同时设置一个或者多个key-value对

> mset key value [key value ...]

* 同时设置一个或者多个key-value对,当且仅当所有给定key 均不存在

> msetnx key value [key value...]

* 将值value关联到key,并设置key的过期时间

> setex key seconds value #设置为seconds(秒计)
>
> psetex key milliseconds value # 以毫秒计

* 只有key不存在时设置key值

> setnx key value

* 对key所存储的value,获取指定偏移量上的位(bit).相当于把value 二进制话,获取对应位置的二进制数据(只有0/1)

> getbit key offset

* 对key所存储的value,设置或请求指定偏移量上的位(只有0/1)

> setbit key offset

* 用value参数覆写给定key所存储的字符串值,从偏移量offset开始.覆写的是原始value值,将新的value从oldvalue指定的Offset位置插入

> setrange key offset value 

```
set test2 redis01
setrange test2 2 abcd # 返回新字符串的长度
get test2 
> reabcd
```

* 如果key已经存在并且是一个字符串,append命令将指定的value追加到该key原来value的结尾,如果不存在,则相当于set key value

> append key value

* 返回key所存储的字符串值的长度

> strlen key

* 对key中存储的数字的操作 value值必须是数字

> incr key
>
> incrby key increment # 将给定key的value加给定的增量值
>
> incrbyfloat key increment #将给定key的value加给定的浮点量值
>
> decr key #将key中存储的value减一
>
> decrby key decrment

### Hash命令

* 赋值

> hmset key field1 value1 [field2 value2] # 同时更新多个值
>
> hset key field value  # 更新单个值
>
> hsetnx key field value # 只有在field不存在的时候,设置hash字段的值,如果存在,则不更新

* 取值

> hget key field # 获取存储在hash表指定字段的值
>
> hgetall key #获取在hash表中指定key的所有字段和值
>
> hkeys key #获取所有hash表中指定key的所有字段(不包含值)
>
> hlen key #获取hash表中字段的数量
>
> hmget key field1 [field2] # 获取所有给定字段的值
>
> hvals key #获取hash表总的所有值

* 查看hash表中指定字段是否存在

> hexists key field #如果存在则返回1,否则返回0

* 删除一个或者多个hash字段

> hdel key field1 [field2...]

* 数值增量操作

> hincrby key field incrment # 整数值新增
>
> hincrbyfloat key field increment # 浮点数新增

* 迭代hash表中的键值对

> hscan key cursor [match pattern] [count count]
>
> hscan hash01 0 match 'abc*' count 100 # match 匹配的是key的过滤,count在小数值范围下没有生效. redis version 6.0.5

### List命令

* 赋值

> lpush key value1[value2...] #将一个或者多个值插入列表头部 先插入的在后面;
>
> lpushx key value # 将一个值插入到已存在列表头部,如果不存在则不会插入
>
> lset key index value #通过索引设置列表元素的值
>
> linsert key before|after pivot value # 在列表的元素前或者后插入元素
>
> rpush key value1 [value2...] #在列表中添加一个或者多个值 先插入的在前面
>
> rpushx key value #将一个值插入到已存在列表的尾部



* 取值

> lindex key index # 通过索引获取列表中的元素
>
> llen key  #获取列表长度
>
> lpop key #移除并获取列表的第一个元素
>
> lrange key start end #获取列表指定范围内的元素

* 移除

> lpop key #移除并获取列表的第一个元素
>
> rpop key #移除列表的最后一个元素
>
> rpoplpush source destination #
>
> lrem key count value # 移除列表元素 count>0 从表头位置搜到表尾;count<0 从表尾绝对值位置搜到表头;count=0 移除表中所有与value相等的值;移除count个数值;如果list中2个值,count =1 or -1的时候,只会删除其中一个,list中还存在一个.value为搜索的内容
>
> blpop key1 [key2] timeout # 移除并获取列表中的第一个元素,如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止. 返回list和第一个元素的值,如果列表没元素则返回nil 和等待时间
>
> brpop key1 [key2] timeout # 移除并获取列表中的最后一个元素如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
>
> brpoplpush source destination timeout # 从列表中弹出一个值将弹出的元素插入到另一个列表中并返回它.如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
>
> ltrim key start top #对一个列表进行修剪,让列表只保留区间内的元素,不在指定区间之内的元素都将被删除



### Set命令

* 赋值

> sadd key member1 [member2...] #向集合添加一个或者多个成员



* 判断member是否是集合key的成员

> sismember key member

* 取值

> smembers key # 返回集合中的所有成员
>
> srandmember key count # 返回集合中一个或者多个随机元素
>
> scan key coursor [match pattern] [count count ]#迭代集合中的元素



* 删除

> spop key #移除并返回集合中一个随机元素
>
> srem key member1 [member2] #移除集合中一个或者多个成员

* 获取集合的成员数

> scard key

* 集合操作

> sdiff key1 [key2...] # 返回给定所有集合的差集
>
> sdiffstore destination key1 [key2..] #返回给定所有集合的差集并存储到destination中
>
> sinter key1 [key2] #返回给定所有集合的交集
>
> sinterstore destination key1 [key2] # 返回给定所有集合的交集并存储到destination中
>
> smember source destination member #将member元素从source集合移动到destination集合
>
> sunion key1 [key2] #返回给定所有集合的并集
>
> sunionstore destionation key1 [key2] #所有给定集合的并集存储在destionation集合中



### Zset命令

* 赋值

> zadd key score1 member1 [score2 member2 ...] #向有序集合添加一个或者多个成员,或者更新已存在成员的score.
>
> zincrby key increment member # 有序集合中对指定成员的分数加上增量increment 
>
> 

* 取值

> zcard key # 获取有序集合的成员数量
>
> zcount key min max # 计算在有序集合中指定区间分数的成员数
>
> zrange key start stop [withscores] # 通过索引区间返回有序集合指定区间内的成员
>
> zrangebyscore key min max [withscores] [limit] # 通过分数返回有序集合指定区间的成员
>
> zrank key member #返回有序集合中指定成员的索引
>
> zrevrange key start stop [withscores] #返回有序集合中指定区间内的成员,通过索引,分数从高到底
>
> zrevrangebyscore key max min [withscores] # 返回有序集合中指定分数区间的成员,分数从高到低排序
>
> zrevrank key member #返回有序集合中指定成员的排名,有序集合按照分数值递减(从大到小)排序
>
> zscore key member # 返回有序集合中成员的分数值

* 删除

> zrem key member [member ...] #移除有序集合中的一个或者多个成员
>
> zremrangebyrank key start stop # 移除有序集合中给定的排名区间的所有成员
>
> zremrangebyscore key min max #移除有序集合中给定的分数区间的所有成员

* 集合操作

> zinterstore destination numkeys key1 [key2 ...] #计算给定的一个或者多个有序集合的交集并将结果存储到新的有序集合中
>
> zunionstore destination numberkey key1 [key2] #计算给定的一个或者多个有序集的并集,并存储在新的key中

* 在有序集合中计算指定字典区间内成员

```
1. zlexcount key min max #返回key相同分数下的指定字典区间内的成员,在有序集合中相同分数的元素之间的顺序是通过字典序排列的.分数必须相同,如果有序集合中的成员分数有不一致的,返回结果就不准
min 字典中排序位置较小的成员,必须以”[“开头,或者以”(“开头,可使用”-“代替
max 字典中排序位置较大的成员,必须以”[“开头,或者以”(“开头,可使用”+”代替

zadd myset 0 a 0 b 0 c 0 d 0 e
zadd myset 0 f 0 g

zlexcount myset - + # (integer) 7
zlexcount myset [b [f # (integer) 5

2. zrangebylex key min max [limit offset count]
分数必须相同,如果有序集合中的成员分数有不一致的,返回结果就不准
limit 返回结果是否分页,指令中包含LIMIT后offset、count必须输入
offset 返回结果起始位置
count 返回结果数量

zrangebylex myset [b [f #返回 b c d e f
zrangebylex myset [b [f limit 1 3 #返回 c d e

3. zremrangebylex key min max # 移除有序集合中给定的字段区间的所有成员
```



### redis hyperLogLog

redis 2.8.9版本添加了hyperLogLog结构

Redis hyperLogLog是用来做基数统计的算法,hyperLogLog的优点是,在输入元素的数量或者体积非常非常大时,计算基数所需的空间总是固定的,并且很小的.

在redis里面,每个HyperLogLog键只需要话费12KB内存,就可以计算2^64个不同元素的基数.这和计算基数时,元素越多越耗费内存就越多的集合形成鲜明对比.

但是,因为HyperLogLog只会根据输入元素来计算基数,而并不会存储元素本身,所以HyperLogLog不能像集合那样,返回输入的各个元素

* 什么是基数

比如数据 {1,3,5,7,5,7,9},那么这个数据集的基数集为{1,3,5,7,9},基数(不重复元素)为5.基数估计就是在误差可以接受的范围内,快速计算基数.

* 常用命令

> pfadd key element [element ...] #添加指定元素到HyperLogLog中
>
> pfcount key [key ...] # 返回给定HyerLogLog的基数估算值
>
> pfmerge destkey sourcekey [sourcekey ...] #将多个HyperLogLog合并成一个HyperLogLog

