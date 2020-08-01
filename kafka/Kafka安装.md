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

* broker.id
* port
* zookeeper.connect
* log.dir
* num.recovery.threads.data.dir
* auto.create.topic.enable

## 主题的默认配置

kafka为新创建的主题提供很多默认配置参数.可以通过管理工具为每个主题单独配置一个部分参数,比如分区个数和数据保留策略.服务器提供的默认配置为基准,它们适用于大部分主题.

* num.partitions
* log.retention.ms. 默认使用log.retention.hours参数来配置时间,默认是168小时.
* log.retention.bytes
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