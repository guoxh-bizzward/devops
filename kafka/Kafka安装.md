# Kafka 安装

## Kafka单机安装

下载JDK

下载zookeeper

下载kafka

```
http://kafka.apache.org/downloads
wget https://mirrors.bfsu.edu.cn/apache/kafka/2.5.0/kafka_2.13-2.5.0.tgz
```

解压

```
tar -zxvf kafka_2.13-2.5.0.tgz
cd kafka_2.13-2.5.0/
```

如果在小内存机器上进行启动,还需要修改kafka-server-start.sh命令里面的内存大小.

```
vi kafka-server-start.sh

if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
fi
```

修改完毕后,启动kafka.,检查端口占用

```
./bin/kafka-server-start.sh -daemon config/server.properties

netstat -tunple|egrep "(2181|9092)"
```

创建并验证主题

```
#创建主题
./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test_topic
#成功提示
Created topic test_topic
#验证主题
./bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test_topic
#成功提示
Topic: test_topic	PartitionCount: 1	ReplicationFactor: 1	Configs: 
	Topic: test_topic	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

发送消息

```
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_topic
>Test Message1
>Test Message2
>D^
```

接收消息

```
kafka消费端
0.90之前的启动方式
./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test_topic --from-beginning
0.90之后的启动方式
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --from-beginning
#输出
Test Message1
Test Message2
```



## broker配置

有一些配置在单机的时候可以直接使用默认值,但是部署到其他环境时需要格外小心.这些参数是单个服务器最基础的配置,它们中的大部分需要经过修改后才能用在集群里

* broker.id 每个broker都需要一个标识符,使用broker.id表示.它的默认值是0,也可以被设定成其他任意整数,这个值在整个kafka集群中必须是唯一的.
* port 默认9092
* zookeeper.connect 用于保存broker元数据的zookeeper地址,默认是localhost:2181.该配置参数是用逗号分隔的一组hostname:port/path列表.其中path是可选的,如果不指定,默认使用根路径
* log.dirs 消息持久化在磁盘的位置.它是一组用逗号分隔的本地文件系统路径.如果指定了多个路径,那么broker会根据"最少使用"原则,把同一分区的日志片段保持在同一个路径下.

注意:broker会往拥有最少数目分区的路径新增分区,而不是往拥有最小磁盘空间的路径新增分区

* num.recovery.threads.data.dir 每个路径的恢复线程数,总线程数=data.dir * log.dirs
* auto.create.topic.enable 是否自动创建主题.

```
默认情况下,kafka会在如下几种情形下自动创建主题
1. 当一个生产者开始往主题写入消息时
2. 当一个消费者开始从主题读取消息时
3. 当任意一个客户端向主题发送元数据请求时.
```



## 主题的默认配置

kafka为新创建的主题提供很多默认配置参数.可以通过管理工具为每个主题单独配置一个部分参数,比如分区个数和数据保留策略.服务器提供的默认配置为基准,它们适用于大部分主题.

* num.partitions 指定新创建的主题将包含多少分区,如果启用了主题自动创建功能(默认开启),主题的分区数就是该参数指定的值,该参数默认为1.

注意: 我们可以增加分区的个数,但是不能减少分区的个数,所以如果要让一个主题的分区数小于num.partitions,需要手动创建该主题.

```
如何选定分区数量
1.主题需要多大的吞吐量

2.从单个分区读取数据的最大吞吐量是多少

3.生产者向单个分区写入数据的吞吐量,生产者的速度一般比消费者快得多,所以最好为生产者多估算一些吞吐量.

4.每个broker包含的分区数,可用磁盘空间和网络带宽

5.如果消息是按照不同的键来写入分区的,那么为已有的主题新增分区就会很困难

6.单个broker对分区个数是有限制的,因为分区越多占用的内存越多,完成首领选举需要的时间越长
```

如果能估算出主题的吞吐量和消费者吞吐量,可以用主题吞吐量除以消费者吞吐量算出分区个数;

如果不知道这些信息,那么根据经验,把分区的大小限制在25GB以内可以得到比较理想的效果.



* log.retention.ms. 决定消息多久以后会被删除.默认使用log.retention.hours参数来配置时间,默认是168小时.还有两个参数log.retention.minutes 和 log.retention.ms,推荐使用log.retention.ms.如果指定了不止一个参数,kafka会优先使用其最小值得那个参数

```
根据时间保留数据和最后修改时间
根据时间保留数据是通过检查磁盘上日志片段文件的最后修改时间来实现的.一般来说,最后修改时间指的就是日志片段的关闭时间,也就是文件里最后一个消息的时间戳.不过,如果使用管理工具在服务器间移动分区,最后修改时间就不准确了.时间误差可以导致这些分区过多的保留数据.
```

* log.retention.bytes 保留消息字节数来判断消息是否过期,作用在每一个分区上.如果一个主题包含8个分区,log.retention.bytes设置为1GB,那么这个主题最多可以保留8GB数据

```

```



* log.retention.ms
* message.max.bytes

## 硬件选择

* 磁盘吞吐量
* 磁盘容量
* 内存
* 网络
* CPU
* 云端kafka

## kafka集群

* 需要多少个kafka
* broker配置
* 操作系统调优
  * 虚拟内存
  * 磁盘
  * 网络

## 生产环境注意事项

* 垃圾回收器选项
* 数据中心布局
* 共享zookeeper