# Spring 解决循环依赖

参考文章 

https://www.jianshu.com/p/6c359768b1dc

https://blog.csdn.net/u010853261/article/details/77940767



Spring容器中两种循环依赖方式

* 构造器循环依赖
* setter方法循环依赖

## Spring循环依赖处理情况

### 构造器注入

**当循环依赖是通过构造器注入的时候，无论这些bean是singleton还是prototype都会报错，无法解决**

表示通过构造器注入构成的循环依赖，此依赖是无解的，强行依赖只能抛出异（BeanCreationException）

### Setter注入

解决方式: Spring容器提前暴露刚完成构造器注入但未完成其他步骤(比如setter注入)的bean完成的.而且只能解决单例作用域的bean循环依赖.通过提前暴露一个单例工厂方法,从而使其他bean能引用到该bean.

### 范围的依赖处理

* 对于prototype作用域bean，spring容器无法完成依赖注入，因为spring容器不进行缓存prototype作用域的bean，因此无法提前暴露一个正在创建中的bean。prototype即原型模式，调用多少次bean，就实例化多少次
* **如果同时有prototype和singleton存在的时候，那么率先获取的bean的作用域是singleton的话，那也是可以通过的**



## 源码解析

Spring单例对象的初始化主要分三步

createBeanInstance实例化 -- 调用对象的构造方法实例化对象

​		|

populateBean填充属性 -- 主要是多bean的依赖属性进行填充

​		|

InitializeBean初始化  -- 调用spring xml的init方法



对于单例来说,在Spring容器整个生命周期中,有且只有一个对象,所以很容易向导整个对象应该存在cache中.Spring为了解决单例的循环依赖问题,使用了三级缓存.

三级缓存的源码

`org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`

```
	/** Cache of singleton objects: bean name to bean instance. */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** Cache of singleton factories: bean name to ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	/** Cache of early singleton objects: bean name to bean instance. */
	private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
```

这三级缓存分别指

* singletonFactories 单例工厂的cache
* earlySingletonObjects 提前曝光的单例对象的cache
* singletonObjects 单例对象cache

我们创建bean的时候,首先想到的是从cache中获取到这个单例的bean,这个缓存就是singletonObjects.主要调用的方法就是`org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`

```java

	/**
	 * Return the (raw) singleton object registered under the given name.
	 * <p>Checks already instantiated singletons and also allows for an early
	 * reference to a currently created singleton (resolving a circular reference).
	 * @param beanName the name of the bean to look for
	 * @param allowEarlyReference whether early references should be created or not
	 * @return the registered singleton object, or {@code null} if none found
	 */
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		// Quick check for existing instance without full singleton lock
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
				synchronized (this.singletonObjects) {
					// Consistent creation of early reference within full singleton lock
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						singletonObject = this.earlySingletonObjects.get(beanName);
						if (singletonObject == null) {
							ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
							if (singletonFactory != null) {
								singletonObject = singletonFactory.getObject();
								this.earlySingletonObjects.put(beanName, singletonObject);
								this.singletonFactories.remove(beanName);
							}
						}
					}
				}
			}
		}
		return singletonObject;
	}
```

* isSingletonCurrentlyInCreation() 判断当前单例bean是否正在创建中,也就是没有初始化完成(比如A的构造器依赖了B对象所以得先去创建B对象,或者在A的populateBean过程中了依赖了B对象,得先去创建B对象,这时的A就是处于创建中的状态)
* allowEarlyReference 是否允许从singletonFactories中通过getObject拿对象

分析`getSingleton()`的整个过程,Spring首先从一级缓存singletonObjects中获取,如果获取不到,并且对象正在创建中,就再去二级缓存earlySingletonObjects中获取.如果还是获取不到且允许从singletonFactories通过getObject()获取,就从三级缓存singletonFactory.getObject()(三级缓存)获取.如果获取到了则进行如下操作

```
this.earlySingletonObjects.put(beanName, singletonObject);
this.singletonFactories.remove(beanName);
```

从singletonFactories中移除,并放入earlySingletonObjects中.其实就是从三级缓存放到二级缓存.

Spring解决循环依赖的诀窍就在于singletonFactories这个cache.这个cache中存的是类型为ObjectFactory,其定义如下

```
@FunctionalInterface
public interface ObjectFactory<T> {

	/**
	 * Return an instance (possibly shared or independent)
	 * of the object managed by this factory.
	 * @return the resulting instance
	 * @throws BeansException in case of creation errors
	 */
	T getObject() throws BeansException;

}
```

在bean创建过程中,有两处比较重要的匿名内部类实现了该接口.

`org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean`

```
if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, () -> {
						try {
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							// Explicitly remove instance from singleton cache: It might have been put there
							// eagerly by the creation process, to allow for circular reference resolution.
							// Also remove any beans that received a temporary reference to the bean.
							destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}
```

`org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean`

```
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
```

此处是解决循环依赖的关键,这段代码发生在createBeanInstance之后,也就是说单例对象此时已经被创建出来了.这个对象已经被生产出来了,虽然还不完美(还没有进行初始化的第二步和第三步),但是已经能被人认出来了(根据对象引用能定位到堆中对象).所以Spring此时将这个对象提前曝光出来让大家认识和使用



这样做有什么好处呢？让我们来分析一下“A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，长大成人，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象也蜕变完美了



作者：LNAmp
链接：https://www.jianshu.com/p/6c359768b1dc
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。