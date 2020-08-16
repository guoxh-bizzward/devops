# MongoDB

## MongoDB安装

MAC

#### 通过brew 安装

```
brew tap mongodb/brew
brew install mongodb-community@4.4
```

安装信息

* 配置文件 /usr/local/etc/mongod.conf
* 日志存储路径 /usr/local/var/log/mongodb
* 数据存储路径 /usr/local/var/mongodb

运行mongodb

```
brew service start mongodb-community@4.4

brew service stop mongodb-community@4.4
```

#### 下载包安装

```
cd /usr/local
sudo curl -O https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-4.4.0.tgz
sudo tar -zxvf mongodb-osx-ssl-x86_64-4.4.0.tgz
sudo mv mongodb-osx-x86_64-4.0.9/ mongodb
```
修改配置文件 `vi ~/.bash_profile`,添加如下内容
> export PATH=/usr/local/mongodb/bin:$PATH

创建日志及数据存放目录
```
sudo mkdir -p /usr/local/var/mongodb
sudo mkdir -p /usr/local/var/log/mongodb

sudo chown user /usr/local/var/mongodb
sudo chown user /usr/local/var/log/mongodb
```
启动mongodb
```
mongod --dbpath /usr/local/var/mongodb --logpath /usr/local/var/log/mongodb/mongo.log --fork
```
--fork 表示在后台运行;
 如果需要使用mongodb的mongorestore等命令还需要下载mongodb-database-tools

```
curl -O https://fastdl.mongodb.org/tools/db/mongodb-database-tools-macos-x86_64-100.1.1.zip
```
下载完将命令放到mongodb的bin下或者将该目录放到环境配置文件中.

## Mongodb命令操作
### insert操作
```
db.<collection>.insertOne(<JSON对象>)
db.<collection>.insertMany([<JSON 1>,<JSON 2>,... <JSON n>])
```
注意,批量操作得时候需要用中括号包裹,相当于数组;
eg
```
db.fruit.insertOne({name:"apple"})
db.fruit.insertMany([{name:"orange"},{name:"pear"},{name:"banana"}])
```

### 查询操作
db.<collection>.find()
find返回得是游标;
eg
```
db.fruit.find();
db.fruit.find({"year":"1975"}); //单条件查询
db.fruit.find({"year":1975","title":"battle"});//多条件and查询
db.fruti.find($and:[{"year":1975","title":"battle"}],{"category":"action"}) //and查询
db.fruit.find($or:{"year":1975},{"title":"batman"});//or查询
db.fruit.find({"title":/^B/}) //正则表达式查找
```

```
a=1 => {a:1}
a<>1 => {a:{$ne:1}}
a>1 => {a:{$gt:1}}
a<1 => {a:{$lt:1}}
a>=1 => {a:{$gte:1}}
a<=1 => {a:{$lte:1}}
a=1 and b=1 => {a:1,b:1} or {$and:[{a:1},{b:1}]}
a=1 or b=1 => {$or:[{a:1},{b:1}]
a is null => {a: {$exists:false}}
a in (1,2,3) => {a: {$in:[1,2,3]}}
```

逻辑运算符

```
$lt:
$lte:
$gt:
$gte:
$ne:
$in:
$nin:
$or:
$and:
```

find搜索子文档
* find 支持使用`find.sub_field` 的形式查询子文档,假设有一个文档:
```
db.fruit.insertOne({
    name:"apple",
    from:{
        country:"CHINA",
        province:"Shandong"
    }
})
```