# SpringBootServletInitializer

使用方法:

```
public class DemoStartUp extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(DemoApplication.class);
    }
}
```

SpringBootServletInitializer的执行过程,简单来说就是通过SpringApplicationBuilder构建并封装SpringApplication对象,并最终调用SpringAppliation的run方法的过程.

SpringBootServletInitializer是原来的web.xml文件的替代

使用嵌入式Servlet,默认是不支持jsp的.SpringBootServletInitializer可以使用外部的Servlet容器,使用步骤

* 必须创建war包,需要创建好web项目的目录;
* 嵌入式tomcat依赖scope指定provided
* 编写SpringBootServletInitializer类子类,并重写configure方法
* 启动服务器

jar包和war包启动区别

* jar 包 执行SpringBootApplication的run方法,先创建IOC容器然后创建嵌入式Servlet容器
* war包 先启动servlet容器,服务启动SpringBoot应用,然后启动IOC容器

SpringBootServletInitializer实例执行onStartup方法的时候通过createRootApplicationContext方法来执行run方法,接下来的过程就同 以jar包形式启动应用的run过程一样,在 内部会创建IOC容器并返回.只是以war包形式的应用在创建IOC容器过程中,不会再创建Servlet容器了.

```
这个地方我们就遇到坑了,我们使用了rpc的方式调用远程的服务,然后在本地启动使用的是jar包的方式启动,访问没有问题,部署到服务器上使用的是外置的tomcat,然后就出问题,注入的DAO是null(此处我们的DAO是通过mybatisplus的方式动态生成的),但是正常通过rest接口调用controller的分页是没有问题,最后发现两者的启动方式不一样,导致servlet容器中没有IOC容器中的bean
```

servlet容器 存放servlet对象,springMVC框架整个是一个servlet对象.

IOC容器在SpringBoot框架中会存放产生servlet容器的工厂,工厂依据主要配置文件以及用户配置的EmbeddedServletContainerCustomizer类对象(主配置文件配置servlet容器底层也是依赖的此编辑器)

Embedded Servlet Container Customizer ： 英译 ：嵌入式 servlet(小服务程序) 容器 编辑器

产生具体的servlet容器， 然后此时，mvc框架的配置生效，向servlet容器注入具体的servlet对象



​简化的说三者运行顺序为：先有IOC容器，然后向IOC容器中注入servlet工厂，再依据该工厂，生成具体的servlet容器，然后容器中导入三大servlet组件：servlet（mvc框架管理的对象，原生servlet等） 、filter、listener