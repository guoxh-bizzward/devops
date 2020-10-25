# Nginx 配置

nginx 提供了如下能力：

* 代理
* Rewrite
* 配置https
* 负载均衡
* ...

## nginx 常用命令

```
nginx -t 
nginx -s reload 
```



## URI资源配置

```

```

## 本地静态文件配置

```

```

## Nginx 总结

### proxy_pass url结尾带不带 /

**proxy_pass**如果以`/`结尾，就相当于是绝对根路径，那么**Nginx**不会把**location**中匹配的路径部分代理走;如果不以`/`结尾，也会代理匹配的路径部分。

示例：后端路径url为 localhost:8091/micro/menu/getmenu；nginx配置端口9999

nginx配置

```
location ^~ /micro {
    proxy_set_header Host $host;
    proxy_pass http://localhost:8091;
}
```

此时访问 localhost:9999/micro/menu/getmenu 就可以看到数据；

nginx配置

```
location ^~ /micro/menu/getmenu {
    proxy_set_header Host $host;
    proxy_pass http://localhost:8091/micro/menu/getmenu/;
}
```

如果走"/"结尾，需要上述配置才能看到数据；

###  location ^~ 作用

`^~` 表示uri以某个常规字符串开头，如果匹配到，则不继续往下匹配。不是正则匹配

### https配置

```
http{
    #http节点中可以添加多个server节点
    server{
        #ssl 需要监听443端口
        listen 443;
        # CA证书对应的域名
        server_name felord.cn;
        # 开启ssl
        ssl on;
        # 服务器证书绝对路径
        ssl_certificate /etc/ssl/cert_felord.cn.crt;
        # 服务器端证书key绝对路径 
        ssl_certificate_key /etc/ssl/cert_felord.cn.key;
        ssl_session_timeout 5m;
        # 协议类型
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        # ssl算法列表 
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        #  是否 服务器决定使用哪种算法  on/off   TLSv1.1 的话需要开启
        ssl_prefer_server_ciphers on;

        location ^~  /api/v1 {
            proxy_set_header Host $host;
            proxy_pass http://192.168.1.9:8080/;
        }
    }
    # 如果用户通过 http 访问 直接重写 跳转到 https 这个是一个很有必要的操作
    server{
        listen 80;
        server_name felord.cn;
        rewrite ^/(.*)$ https://felord.cn:443/$1 permanent;
    }

}
```

### rewrite 配置

rewrite功能让我们的请求到达服务器时重写URI，对请求进行一些预处理

```
#实现如果判断请求为POST的话返回405
location ^~  /api/v1 {
    proxy_set_header Host $host;
    if ($request_method = POST){
      return 405;
    }
    proxy_pass http://192.168.1.9:8080/;
}
```



### 负载均衡

* 最简单的轮询策略

```
http {
    upstream app {
           # 节点1
           server 192.168.1.9:8080;
           # 节点2
           server 192.168.1.10:8081;
           # 节点3
           server 192.168.1.11:8082;
    }
    server {
        listen       80;
        server_name  felord.cn;
    #   ^~ 表示uri以某个常规字符串开头，如果匹配到，则不继续往下匹配。不是正则匹配
        location ^~  /api/v1 {
            proxy_set_header Host $host;
            # 负载均衡
            proxy_pass http://app/;
        }
    }
}
```

* 加权轮询策略 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况

```
upstream app {
       # 节点1
       server 192.168.1.9:8080 weight = 6;
       # 节点2
       server 192.168.1.10:8081 weight = 3;
       # 节点3
       server 192.168.1.11:8082 weight = 1;
}
```

* IP HASH 根据访问ip进行hash，这样每个客户端固定访问服务器，如果服务器宕机，需要手动剔除

```
upstream app {
       ip_hash;
       # 节点1
       server 192.168.1.9:8080 weight = 6;
       # 节点2
       server 192.168.1.10:8081 weight = 3;
       # 节点3
       server 192.168.1.11:8082 weight = 1;
}
```

* 最少连接 请求将转发到连接数较少的服务器上

```
upstream app {
       least_conn;
       # 节点1
       server 192.168.1.9:8080 weight = 6;
       # 节点2
       server 192.168.1.10:8081 weight = 3;
       # 节点3
       server 192.168.1.11:8082 weight = 1;
}

作者：码农小胖哥
链接：https://juejin.im/post/5f184206e51d45348a2b81d4
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

