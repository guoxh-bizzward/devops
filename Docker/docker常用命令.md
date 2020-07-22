# Docker 

## docker架构

docker包含三个基本概念:

### 镜像(image)

```
# 搜索镜像
docker search nginx
# 下载镜像
docker pull nginx
# 登录到docker hub
docker login -u username -p password
# 登出
docker logout
# 上传镜像
docker push name[:tag]
```



```
# 列出本地镜像
docker images
# 删除镜像
docker rmi 镜像id/repository/repository:tag
如果存在多个版本的镜像时,比如本地存在一个latest的tomcat和一个8.5.40的tomcat
docker rmi tomcat 默认会删除latest版本的tomcat
docker rmi tomcat:8.5.40 会删除指定tag版本的tomcat
```





### 容器(container)

```
# 创建一个容器但不启动它
docker create --name nginx_test nginx
# 启动容器
docker run -it -d -p 10080:80 --netowrk test-net --name nginx_test nginx
-p 容器内部端口绑定到特定的主机端口,比如 10080:80
-P 容器内部端口随机映射到主机的高端口
--network 指定网络
# 停止容器
docker stop 容器id(全名或者前五个字符)/容器名称
# 删除容器
docker rm 容器id(全名或者前五个字符)/容器名称
# 查看容器
docker ps 查看正在运行的容器, 加 -a 表示查看所有容器,包括已经停止的
# 启动一个已经停止的容器 
docker start 容器id(全名或者前五个字符)/容器名称
# 进入容器
docker attach 容器id
docker exec -it 容器id /bin/bash
attach与exec的区别是 attach 退出容器的时候容器会停止,用exec则不会停止,所以推荐使用exec
# 创建docker 网络
docker network create -d bridge test-net
-d 参数指定docker 网络类型,网络这块查看后续文章
# 查看docker 网络
docker network ls
# docker dns 配置
全局配置
修改宿主机的/etc/docker/daemon.json,增加
{
	"dns":[
		"114.114.114.114",
		"8.8.8.8"
	]
}
修改完需要重启docker
#检查容器dns是否生效命令
docker run -it --rm ubunntu cat etc/resolv.conf
#手动指定容器配置
docker run -it -rm host_ubuntu --dns 114.114.114.114 --dns-search=test.com ubuntu
如果没有使用--dns --dns-serach 容器会默认使用宿主机的/etc/resolv.conf

# 容器与主机之间的数据拷贝
docker cp [option] container:src_path dest_path|- 
docker cp [option] src_path|- container:dest_path

# 导出容器
docker export 容器id > nginx.tar 注意此处有一个 >
# 导入容器
cat nginx.tar | docker import -test/nginx:v1
docker import url
# 查看端口号除了 docker ps 还可以使用下面的命令
docker port 容器id
# 查看容器内部的标准输出
docker log -f 容器id/容器名称
# 查看容器内部运行的进程
docker top 容器id/容器名称
# 查看docker 底层信息
docker inspect 容器id/容器名称
# 杀掉一个运行中的容器
docker kill -s KILL 容器名 # -s 向容器发送一个信号
# 暂停/恢复容器中的所有进程
docker pause 容器id
docker unpause 容器id
```



### 仓库(reposity)



## 容器互联

如果有多个容器互相连接,推荐使用Docker Compose



## Dockerfile

Dockerfile是一个用来构建镜像的文本文件,文本文件包含了一条条构建镜像所需的指令和说明

```
mkdir -p dockerfile
cd dockerfile
vi Dockerfile
```

> 注意: Dockerfile文件名必须是Dockerfile

编辑Dockerfile

```dockerfile
FROM nginx
WORKDIR /usr/share/nginx/html
RUN echo 'first page'>/usr/share/nginx/html/index.html
```

* FROM 指定基础镜像

* WORKDIR 指定工作目录.用WORKDIR指定的工作目录,会在构建镜像的每一层都存在.WORKDIR 指定的工作目录，必须是提前创建好的

* RUN 用于执行后面跟着的命令行命令,有两种格式

  * RUN <命令行>
  * RUN ["可执行文件","参数1","参数2"]

  > RUN ["./test.php","dev","offline"] 等价于 RUN ./test.php dev offline

> 注意:Dockerfile的指令每执行一次都会在docker上新建一层.所以过多无意义的层,会造成镜像膨胀过大

```dockerfile
FROM centos
RUN yum install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
以上执行会创3层镜像,可以简化为以下格式
FROM centos
RUN yum install wget \
	&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
	&& tar -xvf redis.tar.gz
```



Dockerfile 文件执行

```
docker build -t nginx:test . #注意最后面有个点,表示当前上下文路径
```





