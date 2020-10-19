# SpringBoot Mybatis 

maven 依赖

```
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

关注点

`org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration`



`org.mybatis.spring.mapper.MapperScannerConfigurer`

`org.mybatis.spring.mapper.ClassPathMapperScanner`

`org.mybatis.spring.mapper.MapperFactoryBean`

这三个类是关于使用`ClassPathBeanDefinitionScanner`进行自定义注解加载的.小马哥P161