# Mybatis插件之 拦截器

## 插件(plugins)

Mybatis 允许你在映射语句执行过程中的某一点进行拦截调用.默认情况下,Mybatis允许使用插件来拦截的方法包括

* Executor ( update,query,flushStatements,commit,rollback,getTransaction,close,isClosed)
* ParamterHandler(getParamterObject,setParameters)
* ResultSetHanlder(hanleResultSets,handleOutputOaramters)
* StatementHanlder(prepare,parameterize,batch,update,query)

## @Intercepts

在实现interceptor接口的类声明,使该类注册成为拦截器.

`Signature[] value()` 定义需要拦截哪些类的哪些方法.

## @Signature

```
Class<?> type(); //要拦截的类

String method(); //拦截方法

Class<?>[] args();//方法对应的参数
```

## 拦截器

