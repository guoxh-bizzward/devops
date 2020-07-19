# Redis 发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式,发送者发送消息,订阅者接收消息.

Redis客户端可以订阅任意数量的频道;

```
#客户端
subscribe redischat
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redischat"
3) (integer) 1

#服务端端
publish redischat "redis first message #这个时候客户端就会收到消息
```

常用命令

psubscribe pattern [pattern...] # 订阅一个或者多个符合给定模式的频道

pubsub subcommand [argument [argument]] # 查看订阅与发布状态

publish channel message # 将消息发送到指定频道

punsubscribe [pattern [pattern ...]] #退订所有给定模式的频道

subscribe channel [channel ] #订阅给定的一个或者多个频道信息

unsubscribe [channel [channel ....]] #退订给定的频道

