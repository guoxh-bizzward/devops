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
db.fruti.find({$and:[{"year":1975","title":"battle"}],{"category":"action"}}) //and查询
db.fruit.find({$or:[{"year":1975},{"title":"batman"}]});//or查询
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

查询语句

```
db.fruit.find({"from.country":"CHINA"}) //正确写法

db.fruit.find({"from":{country:"CHINA"}}) //错误写法
```

find搜索数组

* find支持对数组中的元素进行搜索.假设有一个文档

```
db.fruit.insertMany([
	{name:"apple",color:["red","green"]},
	{name:"orange",color:["yellow","green"]}
])
```

查询语句

```
db.fruit.find({color:"red"})

db.fruit.find({$or:[{color:"red"},{color:"yellow"}]})
```

find搜索子文档数组

* find搜索子文档的数组

```
db.moves.insertOne({
	"title":"Raider",
	"film_locations":[
		{"city":"New York","state":"CA","country":"usa"},
		{"city":"Rome","state":"Lazio","country":"Italy"},
		{"city":"Florence","state":"SC","country":"usa"}
	]
})
db.moves.insertOne({
	"title":"Raider",
	"film_locations":[
		{"city":"New York","state":"CA","country":"usa"}
	]
})
```

查询城市是Rome的记录

```
db.moves.find({"film_locations.city":"Rome"})
```

* 在数组中搜索子对象的多个字段时,如果使用`$elemMatch`,它表示必须是同一个子对象满足多个条件.

```
//返回对象里面既有Rome 和usa的对象;返回一条数据;
db.moves.find({
	"film_locations.city":"Rome",
	"film_locations.country":"usa"
})

//返回对象里面子表数组同一条记录满足如下条件;无返回数据;
db.moves.find({
	"film_locations":{
		$elemMatch:{"city":"Rome","country":"usa"}
	}
})
```

控制find返回字段

* find可以指定只返回某些字段
* _id字段必须明确指定不返回,否则默认返回;
* 在MongoDB中这种操作称为投影(projection)
* `db.moves.find({},{_id:0,title:1})`,其中第一个大括号内填写查询条件,第二个控制返回属性; 0表示不返回,1 表示返回;



### 删除文档

删除文档使用remove

* remove命令需要配合查询条件使用;
* 匹配查询条件的文档会被删除;
* 指定一个空文档条件会删除所有文档;

```
db.fruit.remove() //报错
db.fruit.remove({"name":"apple"}) //WriteResult({ "nRemoved" : 1 })
db.fruit.remove({}) //删除所有数据
```



### 更新文档

更新文档使用update

* update操作执行格式 `db.<collection>.update(<查询条件>,<更新字段>)`

```
db.fruit.insertMany([
	{name:"apple"},
	{name:"orange"},
	{name:"banana"},
	{name:"apple"}
])
```

执行更新操作

```
db.fruit.update({name:"apple"},{$set:{from:"CHINA"}})
//WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
==>
b.fruit.updateOne({name:"apple"},{$set:{from:"CHINA"}})
```

使用update更新文档需要注意以下几点

*  使用updateOne表示无论条件匹配多少条,始终只更新第一条;
* 使用updateMany表示条件匹配多少条,就更新多少条;
* updateOne/updateMany方法要求更新条件部分必须具有以下之一,否则报错

```
$set/$unset
$push/$pushAll/$pop 
($push 增加一个对象到数组底部;$pushAll 增加多个对象到数组底部;$pop 从数组底部移除一个对象)
$pull/$pullAll
($pull/$pullAll如果匹配到任意值,从数组中删除相应对象)
$addToSet
($addToSet 如果不存在则增加一个值到数组)
```

比如执行`db.fruit.updateOne({name:"apple"},{from:"CHINA"})` 就会报错;



### 删除集合

* 使用`db.<collection>.drop()`来删除一个集合
* 集合中的全部文档都会被删除
* 集合相关的索引也会被删除

```
db.fruit.drop()
```

### 创建数据库

使用 use dbname的方式创建数据库

`````
use test01 //switched to db test01
show dbs
`````

此时通过`show dbs`是看不到你刚才新建的数据库的,空数据库不会显示，需要向其插入至少一个文档.

### 删除数据库

* 使用 db.dropDatabase()来删除数据库
* 数据库相应文档也会被删除,磁盘空间将会被释放

```
use test
db.dropDatabase()
show collections => show tables
show dbs //db 已经没有test了
```

