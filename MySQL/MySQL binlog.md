# MySQL Binlog

## 什么是二进制日志(binlog)

binlog是记录所有数据库表结构变更(create,alter table等)以及表数据修改(insert,update,delete...)的二进制日志.

binlog不会记录select和show这类的操作.

二进制日志文件包括两类文件:

* 二进制日志索引文件 文件后缀为.index,用于记录所有的二进制文件;
* 二进制日志文件 文件后缀为 .00000*,记录数据库所有的DDL和DML(除了数据查询语句)事件

##　什么是事务日志

innodb事务日志包括redo log和undo log.事务日志的目的是 实例或者介质失败,事务日志文件就能派上用场.

* undo log是指事务开始之前,在操作任何数据之前,首先将需操作的数据备份到一个地方;

undo log 用来回滚行记录到某个版本.事务提交前,undo保存了未提交之前的版本数据,undo中的数据可以作为数据旧版本快照供其他并发事务进行快照读.**是为了实现事务的原子性而出现的产物,在Mysql innodb存储引擎中用来实现多版本并发控制**

* redo log是事务中操作的任何数据,将最新的数据备份到一个地方;

redo log不是随事务的提交而写入的,而是在事务的执行过程中,便开始写入redo中.具体的落盘策略可以进行配置.防止在发生故障的时间点,尚有脏页未写入磁盘.在重启mysql服务的时候,根据redo log进行重做,从而达到事务的未入盘数据进行持久化这一特性.**Redo log是为了实现事务的持久性而出现的产物**





![image-20200714104338897](D:\workspace\devops\imgs\mysql redo log.png)

![image-20200714110605373](D:\workspace\devops\imgs\mysql undo log.png)



## 如何开启MySQL的binlog

```
/etc/my.cnf

log-bin=mysql-bin # 开启binlog
binlog-format=ROW # Row模式
server_id=1 #配置mysql replication需要定义,不能重复
```

### binlog 常用格式

* statement 基于SQL语句的复制(statement-based replication,SBR),每一条会修改数据的sql语句会记录到binlog中;

优点: 不需要记录每一条sql语句与每行的数据变化,日志文件小,减少磁盘IO,提高性能;

缺点: 准确性差,对一些系统函数不能准确复制或者不能复制,如now().uuid()等

* row 记录的是每行实际数据的变更

准确性强,能准确复制数据的变更;

日志文件大,较大的网络IO和磁盘IO

* mixed statement和row模式的混合

准确性强,文件大小适中;

可能会发生主从不一致问题;



现在业内推荐使用的是row模式,准确性高;



## 常见的开源框架原理



### 框架

* mysql-binlog-connector-java
* canal

