# Redis Lua

## EVAL

`EVAL luascript numkeys key [key....] arg [arg ...]`

```
39.105.194.192:6379> set hello helloworld
OK
39.105.194.192:6379> get hello
"helloworld"
39.105.194.192:6379> EVAL "return redis.call('GET',KEYS[1])" 1 hello
"helloworld"
39.105.194.192:6379> EVAL "return redis.call('GET','hello')"
(error) ERR wrong number of arguments for 'eval' command
39.105.194.192:6379> EVAL "return redis.call('GET','hello')" 0
"helloworld"

39.105.194.192:6379> EVAL "return redis.call('no_command')" 0
(error) ERR Error running script (call to f_1e6efd00ab50dd564a9f13e5775e27b966c2141e): @user_script:1: @user_script: 1: Unknown Redis command called from Lua script
39.105.194.192:6379> EVAL "return redis.pcall('no_command')" 0
(error) @user_script: 1: Unknown Redis command called from Lua script

39.105.194.192:6379> eval "return 3.14" 0
(integer) 3
39.105.194.192:6379> eval "return tostring(3.14)" 0
"3.14"


```

## 原子执行



## 脚本管理

`SCRIPT LOAD` 

加载脚本到缓存以达到重复使用,避免多次加载浪费带宽,每一个脚本都会通过sha校验返回唯一字符串标识符,需要配置`EVALSHA`命令来执行缓存后的脚本.

```
39.105.194.192:6379> script load "return 'hello'"
"1b936e3fe509bcbc9cd0664897bbe8fd0cac101b"
39.105.194.192:6379> evalsha 1b936e3fe509bcbc9cd0664897bbe8fd0cac101b 0
"hello"
```

`SCRIPT FLUSH`

 

`SCRIPT EXISTS`



`SCRIPT KILL`



其他一些要点

* lua脚本遇到异常时,已执行逻辑不会回滚
* 尽量不要使用lua提供的具有随机性的函数
* 在lua脚本中不要编码`function`函数
* 在脚本编写中生命的变量全部使用`local` 关键字
* 在集群中使用lua脚本要确保逻辑中所有key都分到相同机器,也就是同一个插槽中,可以采用Redis Hash Tag技术
* lua脚本一定不要包含过于耗时,过于复杂的逻辑