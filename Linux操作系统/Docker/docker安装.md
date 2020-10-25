# docker 安装



## docker 镜像加速

在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）

```
vim /etc/docker/daemon.json

{
	"registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

保存后重启docker

```
systemctl daemon-reload
systemctl restart docker
```

