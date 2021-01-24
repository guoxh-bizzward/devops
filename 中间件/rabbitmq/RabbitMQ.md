# RabbitMQ

AMQP advanced Message Queue Protocol



## 交换器

### Direct

`direct exchange` 需要将一个队列绑定到交换器上,要求该消息与一个特定的路由键完全匹配.这是一个完整的匹配.如果队列绑定到该交换器上要求路由键"test",则只有标记为"test"的消息才能被转发



### Topic

`Topic Exchange` 将路由键和某模式进行匹配,此时队列需要绑定到一个模式上.

使用`#`匹配一个或者多个词;使用`*`匹配仅一个词.



### Fanout

`Fanout Exchange` 不处理路由键.只需要简单的将队列绑定到交换机上.一个发送到交换机上的消息都会被转发到与该交换机绑定的所有队列上.很像子网广播,每台子网内的主机都能获取到一份复制的消息.Fanout交换机转发消息是最快的.



## 持久化消息

```
@RabbitListener(
        bindings = @QueueBinding(
                value = @Queue(value = "${mq.config.queue.error}",autoDelete = "true"),
                exchange = @Exchange(value = "${mq.config.exchange}",type = ExchangeTypes.DIRECT,autoDelete = "true"),
                key = "${mq.config.queue.error.routingkey}"

        )
)
```

`@Queue`和`@Exchange`注解都有`autoDelete`属性,值为布尔类型的字符串.

`@Queue`当所有消费客户端断开连接后,是否自动删除队列,true为删除,false为不删除

`@Exchange` 当所有绑定队列都不再使用时,是否自动删除交换队列.true为删除,false为不删除

当所有消费客户端断开连接时,而我们对rabbitmq消息进行了持久化,那么未被消息的消息存储到Rabbitmq服务器的内存中,如果rabbitmq服务器都关闭时,那么未被消费的数据也会丢失.



## ACK机制

ACK机制是消费者从RabbitMQ收到消息并处理弯沉后,反馈给RabbitMQ,RabbitMQ收到反馈后才将此消息从队列中删除.

消息的ACK机制默认是打开的

如果忘记了ACK,后果很严重.当Consumer退出时候,Message会一直重新分发,然后RabbitMQ会占用越来越多的内存,由于RabbitMQ会长时间运行,因此这个内存泄漏是致命的

可以通过重试次数来规避上述问题

```
#开启重试
spring.rabbitmq.listener.simple.retry.enabled=true
#最大重试次数,包含本身 默认每秒重试
spring.rabbitmq.listener.simple.retry.max-attempts=5
```

