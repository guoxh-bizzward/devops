# SpringBoot  + Activiti6

## 基本配置

pom.xml

```
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>29.0-jre</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.72</version>
        </dependency>

        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-spring-boot-starter-basic</artifactId>
            <version>6.0.0</version>
        </dependency>
```

application.properties

```
server.port=8091
server.compression.enabled=true
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://39.105.194.192:3306/activiti6?useUnicode=utf8&serverTimezone=Asia/Shanghai&nullCatalogMeansCurrent=true
spring.datasource.username=root
spring.datasource.password=Gxh133622

spring.activiti.check-process-definitions=true
spring.activiti.database-schema-update=true
spring.activiti.history-level=full

spring.redis.url=redis://123456@39.105.194.192:6379

mybatis.mapper-locations=classpath:mybatis/mysql/**Mapper.xml
```

启动类需要排除掉`org.activiti.spring.boot.SecurityAutoConfiguration`,因为他会调用`spring-security`的内容,但是pom中没有依赖该包.

```
@SpringBootApplication(exclude = SecurityAutoConfiguration.class)
public class SbVhrApplication {

    public static void main(String[] args) {
        SpringApplication.run(SbVhrApplication.class, args);
    }

}
```

这样项目启动的时候,如果数据库表没有流程表数据就会自动创建.

## 流程部署

流程部署可以通过官方提供的war包启动后设计部署,也可以将在线设计的文件下载下来放在resources/processes目录下通过代码的方式加载执行.

本例介绍第二种方式

```
public String deploy(){
	String deployId = repositoryService.createDeployment()
                .key("hello")
                .name("hello")
                .tenantId("tenant")
                .disableSchemaValidation()
                .disableBpmnValidation()
                .addInputStream("demo.bpmn20.xml",in)
                .deploy()
                .getId();
      System.out.println("deploymentId:"+deployId);
      return deployId;
}
```

## 流程启动

```
    public void start(){
        ProcessInstance processInstance =  runtimeService.startProcessInstanceByKey("helloworld");
        System.out.println(processInstance);
    }

    public List<Task> getTask(String uid){
        List<Task> tasks = taskService.createTaskQuery().taskAssignee(uid).list();
        return tasks;
    }
```

