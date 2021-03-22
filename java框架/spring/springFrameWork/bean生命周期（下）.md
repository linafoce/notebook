## Spring中的“父子”

1. 父子类
2. 父子BeanDefinition
3. 父子BeanFactory
4. 父子ApplicationContext

### 父子类

父子类是Java中的概念，在Spring中，当给某个类创建Bean的过程中，Spring不仅仅会对本类中的属性进行自动注入，同时也会对父类的属性进行自动注入。

### 父子BeanDefinition

父子BeanDefinition是Spring中的概念，Spring在根据BeanDefinition创建Bean的过程中，会先看当前BeanDefinition是否存在父BeanDefinition，如果存在则需要进行合并，合并就是把子BeanDefinition和父BeanDefinition中所定义的属性整合起来（如果存在某个属性在父子BeanDefinition中都存在，那么取子BeanDefinition中的属性）



### 父子BeanFactory

BeanFactory是一个Bean的容器，在Spring中，当我们在使用某个BeanFactory去获取Bean时，如果本BeanFactory中不存在该Bean，同时又有父BeanFactory，那么则会检查父BeanFactory是否存在该Bean，如果也不存在，那么则会创建Bean



### 父子ApplicationContext

父子ApplicationContext和父子BeanFactory类似，子ApplicationContext除开可以使用父ApplicationContext来获取Bean之外，还可以使用父ApplicationContext中其他的东西，比如ApplicationListener





## BeanPostProcessor(Bean的后置处理器)

Bean的后置处理器是指Spring在创建一个Bean的过程中，可以通过后置处理器来干涉Bean的创建过程



一个简单的Bean的生命周期：

1. 推断构造方法(确定使用哪个构造方法来实例化对象)
2. 实例化
3. 填充属性
4. 初始化



Spring在这个基础上，在这4步中的某些"间隙"中增加了扩展点，比如：

1. **BeanPostProcessor**：提供了**初始化前**、**初始化后**
2. **InstantiationAwareBeanPostProcessor**：在BeanPostProcessor的基础上增加了**实例化前**、**实例化后**、**填充属性后**
3. **MergedBeanDefinitionPostProcessor**：在BeanPostProcessor的基础上增加了在实例化和实例化后**之间**的扩展点





加载类

1.获取类加载器

2.读beanclass，如果是class类型，直接拿名字，如果不是转字符串

3.通过全限定名去转class对象，其中还要转成jvm里面的样子[Ljava.lang.String,通过这种符号去查询

4.spring可以自己设置类加载器

一般拿的是默认的类加载器，其实就是appclassLoader, jvm最顶层的类加载器

```java
// 设置自己写的类加载器
application.getBeanFactory.setBeanClassLoader("xxxx")
```

实例化前 InstantiationAwareBeanPostProcessor，**如果这个地方我自己实例化返回了，后面的实例化方法都不执行了，因为已经自己的代码实例化了**

**实例化**

实例化后 InstantiationAwareBeanPostProcessor()。实现这个接口

 ---- 在实例化后，这里可以操作beandefinition,通过实现接口**MergeBeandefinitionPostProcessor**，因为这里面入参有beanDefinition，还有beantype，其实这里面可以自己去属性赋值

**填充属性** 代码在 **populateBean**()方法

填充属性后 InstantiationAwareBeanPostProcessor

**aware回掉** **invokeAware()**方法

主要是执行获取spring参数的方法

- `BeanNameAware`：能够获取bean的名称，即是id
- `BeanFactoryAware`：获取BeanFactory实例
- `ApplicationContextAware`：获取`ApplicationContext`
- `MessageSourceAware`：获取MessageSource

**初始化** invoke()方法

**其中还有一个PostContruct注解** ，这个东西也是在初始化前执行

初始化前 applyPostProcessor，

初始化后,想要做什么事可以通过自己实现接口去实现

```java
public Object @Component
public class LubanBeanPostProcessor implements BeanPostProcessor {

  /**
	 * 实例化的对象，和bean的名字自己去操作
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return
	 * @throws BeansException
	 */
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return null;
	}
  
  /**
	 * 初始化后
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return
	 * @throws BeansException
	 */
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return null;
	}
}
```



