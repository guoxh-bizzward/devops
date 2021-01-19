# Kafka

https://juejin.cn/post/6918898595408117773



https://mp.weixin.qq.com/s?__biz=MzA4MTc4NTUxNQ==&mid=2650520278&idx=1&sn=2493fdf575043975ffd9dcf9a170ab26&chksm=8780bd12b0f734048a87da21c84c4e214c4a8b4619596c0c876b166725a7545cd75baba601d9&scene=21#wechat_redirect



## Kafka应用场景

* 活动跟踪
* 传递消息
* 度量指标和日志记录
* 提交日志
* 流处理



## Kafka生产者

### 创建kafka生产者

创建生产者的一些必选属性

* bootstrap.servers
* key.serializer
* value.serializer

```
    public KafkaProducer<String,String> create(){
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers","39.195.194.192:9092");
        properties.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        properties.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String,String> producer = new KafkaProducer<> (properties);
        return producer;
    }
```

kafka发送消息主要有三种方式

* 发送并忘记(send-and-forget)
* 同步发送
* 异步发送



### 自定义Serializer

