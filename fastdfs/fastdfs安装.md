# fastdfs安装

## 安装libfastcommon

```
wget https://github.com/happyfish100/libfastcommon/archive/V1.0.43.tar.gz
tar -zxvf V1.0.43.tar.gz
cd libfastcommon-1.0.43
./make.sh #执行编译
./make.sh install #安装
```

## 安装fastdfs

```
https://github.com/happyfish100/fastdfs/archive/V6.06.tar.gz
tar -zxvf V6.06.tar.gz
cd fastdfs-6.06
./make.sh
./make.sh install
```

查看可执行命令

>  ls -la /usr/bin/fdfs*

## 配置Tracker服务

进入/etc/fdfs目录,有三个.sample后缀的文件(自动生成的fdfs模板配置文件),通过cp命令拷贝tracer.confg.sample,删除.sample作为正式文件

```
cd /etc/fdfs
cp tracker.conf.sample tracker.conf
```

编辑tracke.conf,修改相关参数,base_path的目录要提前创建好

```
# the base path to store data and log files
base_path=/home/storage/fdfs/tracker
# the tracker server port
port = 22122
# HTTP port on this tracker server
http.server_port = 8989 #nginx配置端口
```

启动tracker ,支持 start\|stop|restart

> /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start

查看tracer启动日志,进入刚刚指定的base_path目录

```
cd /home/storage/fdfs/logs
tail -f trackerd.log
```

查看端口占用情况

> netstat -apn| grep fdfs

## 配置Storage服务

进入/etc/fdfs目录,用cp命令拷贝一份storage.conf.sample,修改为storage.conf

编辑storage.conf文件,修改相关参数

```
base_path=/home/storage/fastdfs/storage #storage 存储data和log的根目录,必须提前创建好
port=23000 # storage端口
group_name=group1 # 默认组名,可以根据实际情况修改
store_path_count=1 #存储路径个数,需要和store_path保持一致
store_path0=/home/storage/fastdfs/storage # 如果为空,则使用base_path
tracker_server=127.0.0.1:22122 #配置storage监听的tracker的ip和port
```

启动storage

> /usr/bin/fdfs_storaged /etc/fdfs/storage.conf start

查看日志,在base_path指定的目录下

> tail -f /home/storage/fastdfs/storage/logs/storaged.log

此时再查看tracker.log的日志,发现已经开始选举,并且作为唯一的tracker被选举为leader

> tail -f /home/storage/fastdfs/tracker/logs/trackerd.log

```
[2020-07-24 11:20:54] INFO - file: tracker_service.c, line: 1603, storage ip: 39.105.194.192, tracker ip by socket: 172.17.185.9, tracker ip by report: 39.105.194.192
[2020-07-24 11:20:54] INFO - file: tracker_relationship.c, line: 468, selecting tracker leader...
[2020-07-24 11:20:54] INFO - file: tracker_relationship.c, line: 487, I am the new tracker leader 39.105.194.192:22122
```

通过monitor查看storage是否绑定成功

> /usr/bin/fdfs_monitor /etc/fdfs/storage.conf

```
subdir count per path = 256
current write server index = 0
current trunk file id = 0

	Storage 1:
		id = 39.105.194.192
		ip_addr = 39.105.194.192  ACTIVE
		http domain = 
		version = 6.06
		join time = 2020-07-24 11:20:53
```



## 安装Nginx和fastdfs-nginx-module模块

下载nginx及fastdfs-nginx-module模块

```
wget http://nginx.org/download/nginx-1.19.1.tar.gz
wget https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.22.tar.gz
```

解压安装

```
tar -zxvf nginx-1.19.1.tar.gz
tar -zxvf V1.22.tar.gz
```

安装依赖的库并安装nginx

```
mv fastdfs-nginx-module-1.22 /usr/local/src/fastdfs-nginx-module-1.22
cd nginx-1.19.1
./configure --prefix=/usr/local/nginx --add-module=/usr/local/src/fastdfs-nginx-module-1.22/src
make
make install
```

整理常见的./configure 报错:

```
# gzip module requires the zlib library
yum install -y zlib-devel
# the HTTP rewrite module requires the PCRE library
yum -y install pcre-devel
```

nginx安装完可以通过如下命令查看nginx安装位置

```
whereis nginx
#nginx: /usr/local/nginx
```

nginx启动

```
cd /usr/local/nginx
./nginx
./nginx -s stop #此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程
./nginx -s quit ##此方式停止步骤是待nginx进程处理任务完毕进行停止
./nginx -s reload 
```

查看nginx版本

```
/usr/local/nginx/sbin/nginx -V

nginx version: nginx/1.19.1
built by gcc 8.3.1 20190507 (Red Hat 8.3.1-4) (GCC) 
configure arguments: --prefix=/usr/local/nginx --add-module=/usr/local/src/fastdfs-nginx-module-1.22/src
```

## 配置nginx和fastdfs-nginx-module

配置mod-fastdfs.conf,并配置到/etc/fdfs文件目录下

```
cd /usr/local/src/fastdfs-nginx-module-1.22/src
cp mod_fastdfs.conf /etc/fdfs/
```

进入/etc/fdfs修改mod-fastdfs.conf

```
base_path=
tracker_server=
url_have_group_name=
storage_server_port=
store_path_count=1
store_path0=
```

配置nginx,增加location

```
cd /usr/local/nginx/conf
vi nginx.conf

location ~/M00 {
	root /home/storage/fastdfs/storage/data;
	ngx_fastdfs_module;
}
```

拷贝fastdfs解压目录中的http.conf和mine.types

```
cd /home/fastdfs/fastdfs-6.06/conf
cp mime.types http.conf /etc/fdfs/
```

## fastdfs常用命令测试

### 上传文件

进入/etc/fdfs目录,拷贝client.conf.sample,删除.sample后缀作为正式文件,修改client.conf的相关配置

上传命令

> /usr/bin/fdfs_upload_file <config_file> <local_filename>
>
> /usr/bin/fdfs_upload_file /etc/fdfs/client.conf nohup.out

```
group1/M00/00/00/J2nCwF8ahxmAQoQFAAJlR3oI7Qk608.out
```

### 下载文件

wget命令

```
wget http://39.105.194.192:8989/group1/M00/00/00/J2nCwF8ahxmAQoQFAAJlR3oI7Qk608.out
```

下载文件

```
/usr/bin/fdfs_download_file <config_file> <file_id> [local_filename]
/usr/bin/fdfs_download_file /etc/fdfs/client.conf group1/M00/00/00/J2nCwF8ahxmAQoQFAAJlR3oI7Qk608.out
```

### 删除文件

```
/usr/bin/fdfs_delete_file <config_file> <file_id>
/usr/bin/fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/00/J2nCwF8ahxmAQoQFAAJlR3oI7Qk608.out
```

