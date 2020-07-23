# 一个MySQL实例有多个Activiti数据库问题

使用SpringBoot + activiti6 搭建审批流项目,数据库使用的是MySQL.且我的数据库下存在多个activiti相关的数据库 schema.

```
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring-boot-starter-basic</artifactId>
    <version>6.0.0</version>
</dependency>
```

配置文件

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/activiti6?useUnicode=utf8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123456

spring.activiti.check-process-definitions=false
spring.activiti.database-schema-update=true
```

因为我的数据库下已经存在了一个activiti7的数据库,所以我这次又新建了一个activiti6的数据库,然后在启动的时候没有自动创建表,而是直接进行了activiti 表的查询,并报了如下的错误

```
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: java.sql.SQLSyntaxErrorException: Table 'activiti6.act_ge_property' doesn't exist
### The error may exist in org/activiti/db/mapping/entity/Property.xml
### The error may involve org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty-Inline
### The error occurred while setting parameters
### SQL: select * from ACT_GE_PROPERTY where NAME_ = ?
### Cause: java.sql.SQLSyntaxErrorException: Table 'activiti6.act_ge_property' doesn't exist
```

然后我就开始跟断点看源码查找问题,发现它走到了如下这段代码`org.activiti.engine.impl.db.DbSqlSession`(中间代码省略了一部分)

```
  public String dbSchemaUpdate() {

    String feedback = null;
    boolean isUpgradeNeeded = false;
    int matchingVersionIndex = -1;

    if (isEngineTablePresent()) {

      PropertyEntity dbVersionProperty = selectById(PropertyEntity.class, "schema.version");
      
      ....
      
    }else {
    	dbSchemaCreateEngine();
    }
```

按预想建的是空白数据库,应该要走else的逻辑,怎么会进到if里面呢,我又继续跟了`isEngineTablePresent`这个方法,很简短,就是看数据库里面表是否存在

```
  public boolean isEngineTablePresent() {
    return isTablePresent("ACT_RU_EXECUTION");
  }
```

继续看`isTablePresent`方法,问题就出在下面这段代码上了,这个tables返回有内容,导致tables.next()为true

```
try {
        tables = databaseMetaData.getTables(catalog, schema, tableName, JDBC_METADATA_TABLE_TYPES);
        return tables.next();
      } finally {
        try {
          tables.close();
        } catch (Exception e) {
          log.error("Error closing meta data tables", e);
        }
      }
```

继续看,`DatabaseMetaData`是一个接口,断点进到了下面`com.zaxxer.hikari.pool.HikariProxyDatabaseMetaData#getTables`,接着调用父类的`com.zaxxer.hikari.pool.ProxyDatabaseMetaData#getTables`方法.

```
public ResultSet getTables(String catalog, String schemaPattern, String tableNamePattern, String[] types) throws SQLException {
        ResultSet resultSet = this.delegate.getTables(catalog, schemaPattern, tableNamePattern, types);
        Statement statement = resultSet.getStatement();
        if (statement != null) {
            statement = ProxyFactory.getProxyStatement(this.connection, statement);
        }

        return ProxyFactory.getProxyResultSet(this.connection, (ProxyStatement)statement, resultSet);
    }
```

`this.delegate.getTables(catalog, schemaPattern, tableNamePattern, types);` 这段代码进到了`com.mysql.cj.jdbc.DatabaseMetaDataUsingInfoSchema`这个类中.

```
@Override
    public ResultSet getTables(String catalog, String schemaPattern, String tableNamePattern, String[] types) throws SQLException {
        String db = getDatabase(catalog, schemaPattern);

        db = this.pedantic ? db : StringUtils.unQuoteIdentifier(db, this.quotedId);
        ...
        
  }
```

继续跟`getDatabase()`方法.该方法实现在父类`com.mysql.cj.jdbc.DatabaseMetaData`中

```
protected String getDatabase(String catalog, String schema) {
        if (this.databaseTerm.getValue() == DatabaseTerm.SCHEMA) {
            return schema == null && this.nullDatabaseMeansCurrent.getValue() ? this.database : schema;
        }
        return catalog == null && this.nullDatabaseMeansCurrent.getValue() ? this.database : catalog;
    }
```

这个时候就看到了schema和catelog的都是null,然后继续跟下去的话sql就变成了如下

```
SELECT TABLE_SCHEMA AS TABLE_CAT, NULL AS TABLE_SCHEM, TABLE_NAME, CASE WHEN TABLE_TYPE='BASE TABLE' THEN CASE WHEN TABLE_SCHEMA = 'mysql' OR TABLE_SCHEMA = 'performance_schema' THEN 'SYSTEM TABLE' ELSE 'TABLE' END WHEN TABLE_TYPE='TEMPORARY' THEN 'LOCAL_TEMPORARY' ELSE TABLE_TYPE END AS TABLE_TYPE, TABLE_COMMENT AS REMARKS, NULL AS TYPE_CAT, NULL AS TYPE_SCHEM, NULL AS TYPE_NAME, NULL AS SELF_REFERENCING_COL_NAME, NULL AS REF_GENERATION FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME LIKE 'ACT_RU_EXECUTION' HAVING TABLE_TYPE IN ('TABLE',null,null,null,null) ORDER BY TABLE_TYPE, TABLE_SCHEMA, TABLE_NAME
```

根据这个sql执行的结果可以看到我的数据库中是存在数据的

![image-20200721213259870](../imgs/image-20200721213259870.png)



然后就可开始看为什么catalog为null呢,然后就继续看`nullDatabaseMeansCurrent`这个属性(其实跟到getDatabase()方法的时候已经进到了mysql-connector-java的类中了).

这个属性在`com.mysql.cj.jdbc.DatabaseMetaData`的定义为

```
protected RuntimeProperty<Boolean> nullDatabaseMeansCurrent;
```

看该类的构造方法可以看到

```
protected DatabaseMetaData(JdbcConnection connToSet, String databaseToSet, ResultSetFactory resultSetFactory) {
        this.conn = connToSet;
        this.session = (NativeSession) connToSet.getSession();
        this.database = databaseToSet;
        this.resultSetFactory = resultSetFactory;
        this.exceptionInterceptor = this.conn.getExceptionInterceptor();
        this.databaseTerm = this.conn.getPropertySet().<DatabaseTerm>getEnumProperty(PropertyKey.databaseTerm);
        this.nullDatabaseMeansCurrent = this.conn.getPropertySet().getBooleanProperty(PropertyKey.nullDatabaseMeansCurrent);
        this.pedantic = this.conn.getPropertySet().getBooleanProperty(PropertyKey.pedantic).getValue();
        this.tinyInt1isBit = this.conn.getPropertySet().getBooleanProperty(PropertyKey.tinyInt1isBit).getValue();
        this.transformedBitIsBoolean = this.conn.getPropertySet().getBooleanProperty(PropertyKey.transformedBitIsBoolean).getValue();
        this.useHostsInPrivileges = this.conn.getPropertySet().getBooleanProperty(PropertyKey.useHostsInPrivileges).getValue();
        this.quotedId = this.session.getIdentifierQuoteString();
    }
```

在`com.mysql.cj.conf.PropertyKey`类中可以看到

```
nullDatabaseMeansCurrent("nullDatabaseMeansCurrent", "nullCatalogMeansCurrent", true), //
```

然后跟到`com.mysql.cj.conf.DefaultPropertySet`

```
    public DefaultPropertySet() {
        for (PropertyDefinition<?> pdef : PropertyDefinitions.PROPERTY_KEY_TO_PROPERTY_DEFINITION.values()) {
            addProperty(pdef.createRuntimeProperty());
        }
    }
```

可以看到`PropertyDefinitions.PROPERTY_KEY_TO_PROPERTY_DEFINITION`

跟到`com.mysql.cj.conf.PropertyDefinitions`类中,果然发现了`nullDatabaseMeansCurrent`的定义.

```
new BooleanPropertyDefinition(PropertyKey.nullDatabaseMeansCurrent, DEFAULT_VALUE_FALSE, RUNTIME_MODIFIABLE,
                        Messages.getString("ConnectionProperties.nullCatalogMeansCurrent"), "3.1.8", CATEGORY_METADATA, Integer.MIN_VALUE),
```

可以看到他的默认值为false.

然后我们在配置文件的数据库链接上加上`&nullCatalogMeansCurrent=true`,然后重新执行程序.发现数据库表插入正常.问题解决.

最后的配置文件:

```
spring.datasource.url=jdbc:mysql://localhost:3306/activiti6?useUnicode=utf8&serverTimezone=Asia/Shanghai&nullCatalogMeansCurrent=true
spring.datasource.username=root
spring.datasource.password=123456

spring.activiti.check-process-definitions=false
spring.activiti.database-schema-update=true
```



参考文章

http://www.bieryun.com/4479.html

https://www.jianshu.com/p/dbeeac29ff27