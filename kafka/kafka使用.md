# Kafka

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

