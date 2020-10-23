# SpringBoot PropertySource 注解

能加载properties文件和xml文件，不支持加载yaml文件，可以通过扩展的方式增加支持

`my.properties`

```
book.name=张三的书
book.code=zhangsan&book
```

```
@Component
@PropertySource(value = "my.properties",encoding = "gbk")
@ConfigurationProperties(prefix = "book")
@Data
public class LoadPropertiesData {
    private String code;
    private String name;
}
```