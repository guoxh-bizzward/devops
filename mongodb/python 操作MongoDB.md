# python 操作MongoDB

## 准备工作

下载python

下载pymongo

windows

> python -m pip install pymongo

*unix

> pip install pymongo

验证pymongo是否安装成功

```
python
>>> import pymongo
>>> pymongo.version 
'3.11.0'
```

编写python文件

###  初始化连接

```
from pymongo import MongoClient

uri="mongodb://127.0.0.1:27017"
client = MongoClient(uri)
print(client)
```

执行`python hello.py`.返回`MongoClient(host=['127.0.0.1:27017'], document_class=dict, tz_aware=False, connect=True)`

### 执行insert操作

```
//插入一条新的用户数据
new_user = {"username":"sa","password":"root1234","email":"sa@local.com"}
result = user_coll.insert_one(new_user)
print(result)
```

### 执行更新操作

```
result=user_coll.update_one({"username":"sa"},{"$set":{"phone":"13100100001"}})
print(result)
```

