# Spring Boot Test

## SpringBoot  Junit4

SpringBoot Junit4写法

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTest{
	...
}
```



## SpringBoot Junit5

SpringBoot Junit5 首先在junit类的引入上需要排除掉`junit-vintage-engine`

```
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
```

测试类去掉了`@RunWith`

```
@SpringBootTest
public class SpringBootTest{
	...
}
```

Mock测试

```
@SpringBootTest
@AutoConfigureMockMvc
public class TestingWebApplicationTest {

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void shouldReturnDefaultMessage() throws Exception {
		this.mockMvc.perform(get("/")).andDo(print()).andExpect(status().isOk())
				.andExpect(content().string(containsString("Hello, World")));
	}
}
```



### SpringBoot 为什么要排除掉junit-vintage-engine

在stackoverflow上搜了下

主要有以下几个答案

```
junit-vintage-engine is used for running JUnit 4 tests; junit-jupiter-engine for JUnit 5 tests.

Presumably since you'll be writing only JUnit 5 tests for a new Spring Boot project, the vintage engine won't be needed, hence the default dependency exclusion in the POM.

Reference:

https://junit.org/junit5/docs/current/user-guide
```



```
junit-vintage-engine :

Used in Junit-4 Testing.
Used to call core classes and annotations.
'Assert' which provide assertion methods for performing test.
'Assume' used to place assumptions.
Use annotation like, @Ignore, @Before, etc...
junit-jupiter-engine :

Used in Junit-5 Testing
Provide some API which helpful to write test cases.
'Assertions' provides utility methods of assertion condition for testing.
'Assumptions' - utility method provide condition based on assumption.
Change the bit of annotation name like, @Disable, @BeforeAll, @BeforeEach, etc...
```

