# SpringBoot多数据源配置

使用mybatis-plus提供额多数据源切换

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.3.1</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
```

配置文件

```yaml
spring:
  application:
    name: spring-boot-dynamic-demo
  datasource:
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false #设置严格模式,默认false不启动. 启动后在未匹配到指定数据源时候会抛出异常,不启动则使用默认数据源.
      datasource:
        master:  #替换成自己的数据库连接
          url: jdbc:mysql://127.0.0.1:3306/dynamic1?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&failOverReadOnly=false&zeroDateTimeBehavior=convertToNull&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true&allowMultiQueries=true&rewriteBatchedStatements=true
          username: root
          password: 123456
          driver-class-name: com.mysql.cj.jdbc.Driver
        slave_1:
          url: jdbc:mysql://127.0.0.1:3306/dynamic2?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&failOverReadOnly=false&zeroDateTimeBehavior=convertToNull&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true&allowMultiQueries=true&rewriteBatchedStatements=true
          username: root
          password: 123456
          driver-class-name: com.mysql.cj.jdbc.Driver
```

