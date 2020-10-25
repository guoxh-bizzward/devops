# MyBatis 延迟加载配置

本例会使用两种配置方式演示延迟加载,一种是通过mybatis-config.xml文件的方式,一种是使用java配置的方式.引入的maven包有

```
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.5</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.21</version>
        </dependency>

        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.13.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.13.3</version>
        </dependency>


        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
        </dependency>
```

## mybatis-config.xml方式

application.properties文件存放的是数据库连接信息

```
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/ds0?useUnicode=true&serverTimezone=Asia/Shanghai
username=root
password=root1234
```

log4j2.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="error">
    <appenders>
        <!-- 输出控制台的配置 -->
        <Console name="Console" target="SYSTEM_OUT">
            <!--             控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
            <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/>
            <!--             输出日志的格式 -->
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </Console>
        </appenders>
    <loggers>
        <root level="trace">
            <appender-ref ref="Console"/>
        </root>
    </loggers>
</configuration>
```



mybatis-config.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="application.properties"></properties>
    <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
    <environments default="develop">
        <environment id="develop">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mapper/AuthorMapper.xml"/>
        <mapper resource="mapper/BlogMapper.xml"/>
        <mapper resource="mapper/PostsMapper.xml"/>
    </mappers>
</configuration>
```

注意,此处的mapper的引入方式是使用的resource方式,所以我们把mapper文件放在了resources目录下的mapper文件夹下

创建表,实体类,DAO和mapper文件

```
@Data
public class Author {

    private String aid;
    private String authorName;
}
```

```
@Data
public class Blog {
    private String bid;
    private String bname;
    private Author author;
}
```

```
public interface AuthorDAO {
    Author selectByPrimaryKey(String id);
}
```

```
public interface BlogDAO {
//    @Select("select * from blog where bid=#{id}")
    Blog selectByPrimaryKey(String id);
}
```

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.example.mybatis.dao.AuthorDAO">
    <resultMap id="BaseResultMap" type="org.example.mybatis.po.Author">
        <result column="aid" property="aid" jdbcType="VARCHAR"/>
        <result column="author_name" property="authorName" jdbcType="VARCHAR"/>
    </resultMap>

    <select id="selectByPrimaryKey" parameterType="java.lang.String" resultMap="BaseResultMap">
        select * from author where aid=#{id}
    </select>
</mapper>
```

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.example.mybatis.dao.BlogDAO">
    <resultMap id="BaseResultMap" type="org.example.mybatis.po.Blog">
        <result property="bid" jdbcType="VARCHAR" column="bid"/>
        <result property="bname" jdbcType="VARCHAR" column="bname"/>
        <association property="author" column="aid" javaType="org.example.mybatis.po.Author"
                     select="org.example.mybatis.dao.AuthorDAO.selectByPrimaryKey">
        </association>
    </resultMap>
    <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="string">
        select * from blog where bid=#{bid}
    </select>
</mapper>
```

测试

```
@Test
public void lazyLoadBlog() throws InterruptedException {
    SqlSession sqlSession = null;
    try(InputStream is = Resources.getResourceAsStream("mybatis-config.xml")){
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
    sqlSession = sqlSessionFactory.openSession();
    }catch (IOException e){

    }
    BlogDAO blogDAO = sqlSession.getMapper(BlogDAO.class);
    Blog blog = blogDAO.selectByPrimaryKey("1001");
    TimeUnit.SECONDS.sleep(5);
    Author author = blog.getAuthor();
    System.out.println(author.getAuthorName());
}
```

日志:

```
17:24:02.375 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==>  Preparing: select * from blog where bid=?
17:24:02.429 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==> Parameters: 1001(String)
17:24:02.482 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==    Columns: bid, bname, aid
17:24:02.483 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==        Row: 1001, 第一篇文章, 1
17:24:02.616 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - <==      Total: 1
17:24:07.622 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==>  Preparing: select * from author where aid=?
17:24:07.623 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==> Parameters: 1(String)
17:24:07.625 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==    Columns: aid, author_name
17:24:07.626 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==        Row: 1, 张三
17:24:07.627 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - <==      Total: 1
张三
```



## Java配置方式

java配置方式和使用mybatis-config.xml的方式有一些不同.

* mybatis-config.xml的mapper可以通过resource标签或者url标签指定位置,mapper文件和接口可以不在一个文件夹下,但是如果mapper使用的是class方式的必须要和接口同包且文件名必须相同;
* 懒加载的设置位置必须在configuration定义之后,在其他地方配置无效

将resources目录下的mapper文件复制到dao目录下,并修改名字和接口名一致

配置configuration

```
    public SqlSession init(){
        SqlSession sqlSession = null;

        PooledDataSource dataSource = new PooledDataSource();
        dataSource.setDriver(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);

        TransactionFactory transactionFactory = new JdbcTransactionFactory();
        Environment environment = new Environment("development",transactionFactory,dataSource);
        Configuration configuration = new Configuration(environment);
        configuration.setLazyLoadingEnabled(true);
        configuration.setAggressiveLazyLoading(false);
        
        configuration.addMapper(AuthorDAO.class);
        configuration.addMapper(BlogDAO.class);
        

        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
//        sqlSessionFactory.getConfiguration().setLazyLoadingEnabled(true);
//        sqlSessionFactory.getConfiguration().setAggressiveLazyLoading(false);

        sqlSession = sqlSessionFactory.openSession();
        return sqlSession;
    }

    @Test
    public void lazyLoadBlog() throws InterruptedException {
        SqlSession sqlSession = init();
//        sqlSession.getConfiguration().setAggressiveLazyLoading(false);
//        sqlSession.getConfiguration().setLazyLoadingEnabled(true);
        BlogDAO blogDAO = sqlSession.getMapper(BlogDAO.class);
        Blog blog = blogDAO.selectByPrimaryKey("1001");
        TimeUnit.SECONDS.sleep(5);
        Author author = blog.getAuthor();
        System.out.println(author.getAuthorName());
    }
```

日志:

```
17:38:36.679 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==>  Preparing: select * from blog where bid=?
17:38:36.762 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==> Parameters: 1001(String)
17:38:36.807 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==    Columns: bid, bname, aid
17:38:36.813 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==        Row: 1001, 第一篇文章, 1
17:38:36.952 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - <==      Total: 1
17:38:41.961 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==>  Preparing: select * from author where aid=?
17:38:41.962 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - ==> Parameters: 1(String)
17:38:41.967 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==    Columns: aid, author_name
17:38:41.968 TRACE org.apache.ibatis.logging.jdbc.BaseJdbcLogger 143 trace - <==        Row: 1, 张三
17:38:41.969 DEBUG org.apache.ibatis.logging.jdbc.BaseJdbcLogger 137 debug - <==      Total: 1
张三
```



## 部分源码解读

###  为什么mybatis-config.xml中的mapper使用class方式引入和通过configuration.addMapper()的方式必须要和DAO同级? 

* org.apache.ibatis.session.Configuration#addMapper

```
  public <T> void addMapper(Class<T> type) {
    mapperRegistry.addMapper(type);
  }
```

* org.apache.ibatis.binding.MapperRegistry#addMapper

```
public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
      if (hasMapper(type)) {
        throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
      }
      boolean loadCompleted = false;
      try {
        knownMappers.put(type, new MapperProxyFactory<>(type));
        // It's important that the type is added before the parser is run
        // otherwise the binding may automatically be attempted by the
        // mapper parser. If the type is already known, it won't try.
        MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
        parser.parse();
        loadCompleted = true;
      } finally {
        if (!loadCompleted) {
          knownMappers.remove(type);
        }
      }
    }
  }
```

* 看parser.parse();这个方法 org.apache.ibatis.builder.annotation.MapperAnnotationBuilder#parse

```
public void parse() {
    String resource = type.toString();
    if (!configuration.isResourceLoaded(resource)) {
      loadXmlResource();
      configuration.addLoadedResource(resource);
      assistant.setCurrentNamespace(type.getName());
      parseCache();
      parseCacheRef();
      for (Method method : type.getMethods()) {
        if (!canHaveStatement(method)) {
          continue;
        }
        if (getAnnotationWrapper(method, false, Select.class, SelectProvider.class).isPresent()
            && method.getAnnotation(ResultMap.class) == null) {
          parseResultMap(method);
        }
        try {
          parseStatement(method);
        } catch (IncompleteElementException e) {
          configuration.addIncompleteMethod(new MethodResolver(this, method));
        }
      }
    }
    parsePendingMethods();
  }
```

* 有一个loadXmlResource()方法

```
private void loadXmlResource() {
    // Spring may not know the real resource name so we check a flag
    // to prevent loading again a resource twice
    // this flag is set at XMLMapperBuilder#bindMapperForNamespace
    if (!configuration.isResourceLoaded("namespace:" + type.getName())) {
      String xmlResource = type.getName().replace('.', '/') + ".xml";
      // #1347
      InputStream inputStream = type.getResourceAsStream("/" + xmlResource);
      if (inputStream == null) {
        // Search XML mapper that is not in the module but in the classpath.
        try {
          inputStream = Resources.getResourceAsStream(type.getClassLoader(), xmlResource);
        } catch (IOException e2) {
          // ignore, resource is not required
        }
      }
      if (inputStream != null) {
        XMLMapperBuilder xmlParser = new XMLMapperBuilder(inputStream, assistant.getConfiguration(), xmlResource, configuration.getSqlFragments(), type.getName());
        xmlParser.parse();
      }
    }
  }
```

通过这地方可以看到通过class方式加载的时候xml文件找到是同包下的同名xml文件



### 为什么在mvn clean package之后在包路径内找不到xml文件

这个需要修改pom.xml文件的build里面的配置

```
    <build>
        <finalName>mybatis-example</finalName>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*</include>
                </includes>
            </resource>
        </resources>
    </build>
```

添加以后再执行mvn clean package就会发现在包路径里面有了xml文件.