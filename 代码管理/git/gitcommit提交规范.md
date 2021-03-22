# git commit 提交规范

提交格式

```xml
 <type>(<scope>): <subject>
 <BLANK LINE>
 <body>
 <BLANK LINE>
 <footer>
```

## 提交类型 type

* feat 功能有变更的时候都可以采用这种类型的type
* fix bug修复
* docs 更新了文档或者更新了注释
* style 代码格式调整,比如执行了format,更改了tab显示等
* refactor 重构代码,指的是代码结构的调整,比如使用了一些设计模式重新组织了代码
* perf 对项目或者模块进行了性能优化,比如一些jvm参数改动
* test 增加了单元测试和自动化相关的代码
* build 影响编译的一些修改,比如更改了maven插件,增加了npm的过程等;
* ci 持续集成方面的更改.
* chore 其他改动.比如一些注释修改或者文件清理.不影响src和test代码文件的,都可以放在这里
* Revert  回滚一些前面的代码

## 范围scope

主要是代码的影响面.scope没有要求强制,但团队可以按照自己的理解进行设计.通常由技术维度和业务维度两种划分方式.比如按照技术分为 controller,dto,service,dao等;但因为一个功能提交会涉及稻草多个scope,所以按照技术维度分的情况很少;

按照业务模块划分,比如按照user,order等划分,可以很容易的看出是影响用户模块还是order模块;

## 其他

### 主题subject

总体概括

### 正文body

详细的改动记录

### 尾部footer

添加一些额外的hook,比如提交记录之后,自动关闭jira的工单(jira和gitlab是可以联动的)

### Skip CI

忽略自动构建