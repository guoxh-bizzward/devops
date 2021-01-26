# Spring Bean生命周期

SpringBean 生命周期包括4类

* bean自身方法 包括bean本身调用的方法和配置文件中配置的 `<bean>`的`init-method`和`destroy-method`指定的方法
* bean声明周期接口方法 包括BeanNameAware,BeanFactoryAware,InitalizingBean和DiposableBean这些接口方法
* 容器生命周期接口方法 包括了 InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现,一般称它们为 后处理器
* 工厂后处理接口方法 包括了AspectJWeavingEnabler,ConfigurationClassPostProcessor,CustomAutowireConfigurer等非常有用的工厂后处理器.工厂后处理器也是容器级的.在应用上下文装配配置文件之后立即调用.



实例化BeanFactoryPostProcessor实现类

​             |

执行BeanFactoryPostProcessor的postProcessBeanFactory方法

​			 |

实例化BeanPostProcessor实现类

​			 |

实例化InstantiationAwareBeanPostProcessorAdapter实现类

​			 |

执行InstantiationAwareBeanPostProcessorAdapter的postProcessBeforeInstantiation方法

​             |

执行bean的构造器

​			 |

执行InstantiationAwareBeanPostProcessorAdapter的postProcessPropertyValues方法

​			 |

为bean注入属性

​			 |

调用BeanNameAware的setName方法

​			 |

调用BeanFacotryAware的setBeanFacotry方法

​			 |

执行BeanPostProcessor的postProcessBeforeInitialzation方法

​			 |

调用InitializingBean的afterPropertiesSet方法

​			 |

调用<bean>的init-method属性指定的初始化方法

​			 |

执行BeanPostProcessor的postProcessAfterInitialization方法

​			 |

执行InstantiationAwareBeanPostProcessor的postProcessAfterInitialization方法

​			 |

容器初始化成功,执行正常调用,下面销毁容器

​			 |

调用DisposibleBean的destory方法

​			 |

调用<bean>的destory-method属性指定的初始化方法



```
现在开始初始化容器
15:59:57.416 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@7e2d773b
15:59:57.664 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 4 bean definitions from class path resource [bean.xml]
15:59:57.706 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'beanFactoryPostProcessor'
这是BeanFactoryPostProcessor实现类构造器
BeanFactoryPostProcessor调用postProcessBeanFactory方法
15:59:57.728 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'beanPostProcessor'
这是beanprocessor 实现类构造器
15:59:57.728 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'instantiationAwareBeanPostProcessor'
这是InstantiationAwareBeanPostProcessorAdapter实现类构造器
15:59:57.734 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'person'
InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法
[构造器]调用person的构造器实例化
InstantiationAwareBeanPostProcessor调用postProcessProperties方法
[属性注入]注入address属性
[属性注入]注入name属性
[属性注入]注入phone属性
[BeanNameAware]调用BeanNameAware.setBeanName
[BeanFactoryAware]调用BeanFactoryAware.setBeanFactory
BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行修改
[InitializingBean]调用InitializingBean.afterPropertiesSet
init-method 调用bean的init-method方法
BeanPostProcessor接口方法postProcessAfterInitialization对属性进行修改
InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法
容器初始化成功
Person{name='张三', address='广州', phone=110, beanFactory=org.springframework.beans.factory.support.DefaultListableBeanFactory@6ed3ccb2: defining beans [beanPostProcessor,instantiationAwareBeanPostProcessor,beanFactoryPostProcessor,person]; root of factory hierarchy, beanName='person'}
现在开始关闭容器
15:59:57.813 [SpringContextShutdownHook] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Closing org.springframework.context.support.ClassPathXmlApplicationContext@7e2d773b, started on Tue Jan 26 15:59:57 CST 2021
[DisposableBean]调用DisposableBean.destroy
destroy-method 调用bean的destroy-method方法
```

