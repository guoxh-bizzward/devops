# Redis 哨兵配置

下载redis,本例以最新的稳定版redis 6.0.8为例介绍;

下载redis 安装包;解压并编译;

```
wget http://download.redis.io/releases/redis-6.0.8.tar.gz
tar -zxvf redis-6.0.8.tar.gz
cd redis-6.0.8
make 
mkdir -p ../redis01 ../redis02 ../redis03
#将redis.conf sentinel.conf 复制到上述三个文件夹中
...

cp redis-cli redis-server redis-sential ../redis01
```

## Redis 主从配置

修改redis01/redis02/redis03下的redis.conf文件

```
## redis01/redis.conf 主服务
1. 将bind 127.0.0.1 注释掉;
2. protected-mode yes 改成 no;
3. port 6379
4. pidfile 不调整
5. 添加密码配置 requirepass 123456
6. 添加密码后,需要配置主从同步密码,和redis密码要一致; masterauth 123456

## redis02/redis.conf 从服务
1. 将bind 127.0.0.1 注释掉;
2. protected-mode yes 改成 no;
3. port 6379
4. pidfile /var/run/redis_6380.pid
5. 添加密码配置 requirepass 123456
6. 添加密码后,需要配置主从同步密码,和redis密码要一致; masterauth 123456
7. 添加 slaveof 127.0.0.1 6379

## redis03/redis.conf 从服务
1. 将bind 127.0.0.1 注释掉;
2. protected-mode yes 改成 no;
3. port 6379
4. pidfile /var/run/redis_6381.pid
5. 添加密码配置 requirepass 123456
6. 添加密码后,需要配置主从同步密码,和redis密码要一致; masterauth 123456
7. slaveof 127.0.0.1 6379

```

配置完毕后,启动redis

```
./redis01/redis-server redis01/redis.conf
./redis01/redis-server redis02/redis.conf
./redis01/redis-server redis03/redis.conf

./redis01/redis-cli -h 127.0.0.1 -p 6379 -a 123456
> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=299028,lag=0
slave1:ip=127.0.0.1,port=6381,state=online,offset=299028,lag=0
master_replid:f931a00edf217702e8a597198b5ae81a44a44fad
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:299028
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:299028
```



## Redis 哨兵配置

修改三个文件夹下的sentinel.conf配置

```
## redis01/sential.conf
port 26379
sentinel monitor mymaster 127.0.0.1 6379 2 # 这段要放在所有配置的最前面
sentinel auth-pass mymaster 123456

## redis02/sential.conf
port 26382
sentinel monitor mymaster 127.0.0.1 6379 2 # 这段要放在所有配置的最前面
sentinel auth-pass mymaster 123456

## redis03/sential.conf
port 26381
sentinel monitor mymaster 127.0.0.1 6379 2 # 这段要放在所有配置的最前面
sentinel auth-pass mymaster 123456
```



## Springboot redis 哨兵配置

```
## redis 哨兵配置
spring.redis.sentinel.master=mymaster
spring.redis.sentinel.nodes=127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381
spring.redis.password=123456
#spring.redis.sentinel.password=123456
```

