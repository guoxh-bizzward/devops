# MongoDB 聚合查询

## mongoDB 聚合框架

MongoDB聚合框架(Aggregation Framework)是一个计算框架,可以做以下操作

* 作用在一个或者多个集合上
* 对集合中的数据进行一系列运算
* 将这些数据转换为期望的形式

从效果而言,聚合框架相当于SQL查询中的 group by,left join,as 等;

### 管道(pipeline) 和步骤(stage)

整个聚合运算过程称为管道,它是由多个步骤组成.

每个管道:

* 接收一系列文档(原始数据)
* 每个步骤对这些文档进行一系列运算
* 结果文档输出给下一个步骤

```
原始数据 =>(步骤1)=>中间结果1=>(步骤2)=>中间结果2=>...=>最终结果
```

聚合运算的基本格式

```
pipeline=[$stage1,$stage2,...$stageN];

db.<collection>.aggregate(
	pipeline,
	{options}
)
```

常见步骤

```
$match => 过滤 => sql的where
$project => 投影 => AS
$sort => =>order by
$group => =>group by
$skip/$limit => 结果限制 => SKIP/LIMIT
$lookup => 左外连接 => left outer join

$unwind => 展开数组 =>N/A
$graphLookup => 图搜索 => N/A
$facet/$bucket => 分面搜索=>N/A
```

常见步骤中的运算符

```
$match
> $eq $gt $gte $lt $lte 
> $and $or $not $in
> $getWithin $intersect
> ....
$project 选择需要或者不需要的字段
> $map $reduce $filter
> $range
> $multiply $divide $substract $add
> $year $month $dayOfMonth $hour $minute $second
> ....
$group
> $sum $avg
> $push $addToset
> $first $last $max $min
> ....
```

聚合运算的场景

聚合运算用于OLAP 和 OLTP 场景

```
OLTP on-line transaction processing  联机事务处理
> 计算
OLAP on-line analytical processing 联机分析处理
> 分析一段时间内的销售总额,均值
> 计算一段时间内的净利润
> 分析购买人的年龄分布
> 分析学生成绩分布
> 统计员工绩效
```

MQL常用步骤和SQL对比

```
SQL
select first_name AS '名',last_name AS '姓' from users where gender='男' limit 100 skip 20

=>MQL
db.users.aggregate([
	{$match:{"gender":"男"}},
	{$limit:100},
	{$skip:20},
	{$project:{"名":"$fist_name","姓":"$last_name"}}
])

SQL 
select department,count(NULL) as emp_qty from users where gender='女' group by department having count(*)<10

=>MQL
db.users.aggregate([
	{$match:{"gender":"女"}},
	{$group:{
		_id:"$DEPARTMENT",
		emp_qty:{$sum:1}
	}},
	{$match:{emp_qty:{$lt:10}}}
])
```



```
$unwind


$bucket 


$facet/$bucket

```



## 聚合操作

