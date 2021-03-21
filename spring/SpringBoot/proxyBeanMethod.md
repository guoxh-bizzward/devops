# proxyBeanMethod

该属性为spring中的`@Configuration`注解的参数

```
	/**
	 * Specify whether {@code @Bean} methods should get proxied in order to enforce
	 * bean lifecycle behavior, e.g. to return shared singleton bean instances even
	 * in case of direct {@code @Bean} method calls in user code. This feature
	 * requires method interception, implemented through a runtime-generated CGLIB
	 * subclass which comes with limitations such as the configuration class and
	 * its methods not being allowed to declare {@code final}.
	 * <p>The default is {@code true}, allowing for 'inter-bean references' via direct
	 * method calls within the configuration class as well as for external calls to
	 * this configuration's {@code @Bean} methods, e.g. from another configuration class.
	 * If this is not needed since each of this particular configuration's {@code @Bean}
	 * methods is self-contained and designed as a plain factory method for container use,
	 * switch this flag to {@code false} in order to avoid CGLIB subclass processing.
	 * <p>Turning off bean method interception effectively processes {@code @Bean}
	 * methods individually like when declared on non-{@code @Configuration} classes,
	 * a.k.a. "@Bean Lite Mode" (see {@link Bean @Bean's javadoc}). It is therefore
	 * behaviorally equivalent to removing the {@code @Configuration} stereotype.
	 * @since 5.2
	 */
	boolean proxyBeanMethods() default true;
```

## 解释

`proxyBeanMethod = true` 或者不写 默认为 true,是 full模式

`proxyBeanMethod = false` 是lite模式

不带`@Configuration`的类叫 lite 配置类.

`org.springframework.context.annotation.ConfigurationClassUtils#checkConfigurationClassCandidate`

```
Map<String, Object> config = metadata.getAnnotationAttributes(Configuration.class.getName());
if (config != null && !Boolean.FALSE.equals(config.get("proxyBeanMethods"))) {
beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_FULL);
}
else if (config != null || isConfigurationCandidate(metadata)) {
beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_LITE);
}
else {
return false;
}
```

Full模式下通过方法调用指向的仍然是原来的bean;

利用cglib代理增强,bean是单例的,@Bean方法调用生成实例时,如果已经存在了这个bean,直接返回

`org.springframework.context.annotation.ConfigurationClassPostProcessor#enhanceConfigurationClasses`

```
	if (ConfigurationClassUtils.CONFIGURATION_CLASS_FULL.equals(configClassAttr)) {
				if (!(beanDef instanceof AbstractBeanDefinition)) {
					throw new BeanDefinitionStoreException("Cannot enhance @Configuration bean definition '" +
							beanName + "' since it is not stored in an AbstractBeanDefinition subclass");
				}
				else if (logger.isInfoEnabled() && beanFactory.containsSingleton(beanName)) {
					logger.info("Cannot enhance @Configuration bean definition '" + beanName +
							"' since its singleton instance has been created too early. The typical cause " +
							"is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor " +
							"return type: Consider declaring such methods as 'static'.");
				}
				configBeanDefs.put(beanName, (AbstractBeanDefinition) beanDef);
			}
		}
```

lite模式下,直接返回新实例对象

Spring5.2.0版本之后,建议配置类均采用Lite模式,即显式设置`proxyBeanMethod=false`,SpringBoot 2.2.0版本之后就把他的所有的自动配置类的此属性都改成false.以提供spring启动速度.

## 注意

如果在@Configuration类中存在的一个方法被注册为bean,另一个方法注册为bean的时候引用到了这个方法,则此处的`proxyBeanMethod`必须为true,否则编译的时候都会报错.

```
有了 proxyBeanMethods 属性后，配置类不会被代理了。主要是为了提高性能，如果你的 @Bean 方法之间没有调用关系的话可以把 proxyBeanMethods 设置为 false。否则，方法内部引用的类生产的类和 Spring 容器中类是两个类。

前面说了加了 proxyBeanMethods 之后不会被代理了，所以配置类和方法也可以使用 final 修饰了，5.2之前的版本是强制校验的。

作者：AlanSun2
链接：https://www.jianshu.com/p/0bf8d3acf421
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

