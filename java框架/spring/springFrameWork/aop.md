# Spring中的AOP底层原理分析

## AOP中的几个概念



### Advisor 和 Advice



Advice，我们通常都会把他翻译为通知，其实很不好理解，其实他还有另外一个意思，就是“建议”，我觉得把Advice理解为“建议”会更好。



比如，我们已经完成了一个功能，这时客户跟我们说，我建议在这个功能之前可以再增加一些逻辑，再之后再增加一些逻辑。



在Spring中，Advice分为：

1. 前置Advice：MethodBeforeAdvice
2. 后置Advice：AfterReturningAdvice
3. 环绕Advice：MethodInterceptor
4. 异常Advice：ThrowsAdvice



在利用Spring AOP去生成一个代理对象时，我们可以设置这个代理对象的Advice。

而对于Advice来说，它只表示了“建议”，它没有表示这个“建议”可以用在哪些方面。

就好比，我们已经完成了一个功能，客户给这个功能提了一个建议，但是这个建议也许也能用到其他功能上。

这时，就出现了Advisor，表示一个Advice可以应用在哪些地方，而“哪些地方”就是Pointcut（切点）。



### Pointcut

切点，表示我想让哪些地方加上我的代理逻辑。

比如某个方法，

比如某些方法，

比如某些方法名前缀为“find”的方法，

比如某个类下的所有方法，等等。

在Pointcut中，有一个MethodMatcher，表示方法匹配器。



### 基本用法

```java
public static void main(String[] args) {
//		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
//		LubanService lubanService = applicationContext.getBean("lubanService", LubanService.class);
//
//		lubanService.test();

		ProxyFactory factory = new ProxyFactory();
		// 目标对象,就是实现类
    // 最好设置，否则他会校验，抱空指针
		factory.setTarget(new LubanService());

		factory.addAdvice(new AfterReturningAdvice(){

			@Override
			public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
				System.out.println(method.getName() + "方法返回后");
			}
		});
  
  	factory.addAdvice(new MethodBeforeAdvice() {
			@Override
			public void before(Method method, Object[] args, Object target) throws Throwable {
				System.out.println("执行方法前");
				method.invoke(target,args);
			}
		});

		LubanService userService = (LubanService) factory.getProxy();
		userService.test();


	}
```







## 使用ProxyFactory通过编程创建AOP代理



定义一个MyAdvisor

```java
public class MyAdvisor implements PointcutAdvisor {

    @Override
    public Pointcut getPointcut() {
        NameMatchMethodPointcut methodPointcut = new NameMatchMethodPointcut();
        methodPointcut.addMethodName("test");
        return methodPointcut;
    }

    @Override
    public Advice getAdvice() {
        MethodBeforeAdvice methodBeforeAdvice = new MethodBeforeAdvice() {
            @Override
            public void before(Method method, Object[] args, Object target) throws Throwable {
                System.out.println("执行方法前"+method.getName());
            }
        };

        return methodBeforeAdvice;
    }

    @Override
    public boolean isPerInstance() {
        return false;
    }
}
```



定义一个UserService

```java
public class UserService {

    public void test() {
        System.out.println("111");
    }
}
```



```java
ProxyFactory factory = new ProxyFactory();
factory.setTarget(new UserService());
factory.addAdvisor(new MyAdvisor());
UserService userService = (UserService) factory.getProxy();
userService.test();
```

## ProxyFactory的工作原理

### 方法创建

```java
public ProxyFactory(Object target) {
		setTarget(target);
		setInterfaces(ClassUtils.getAllInterfaces(target));
	}
```

## TargetSource

1. singletonTargetSource 如果赋值了就是这个, targetclass也是有值的

2. EmptyTargetSource 没有值,targetClass也是有值的可以setTargetClass

```java
public void setTarget(Object target) {
		setTargetSource(new SingletonTargetSource(target));
	}
```



ProxyFactory就是一个代理对象生产工厂，在生成代理对象之前需要对代理工厂进行配置。

ProxyFactory在生成代理对象之前需要决定到底是使用JDK动态代理还是CGLIB技术：



```java
// config就是ProxyFactory对象

// optimize为true,或proxyTargetClass为true,或用户没有给ProxyFactory对象添加interface
if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
    Class<?> targetClass = config.getTargetClass();
    if (targetClass == null) {
        throw new AopConfigException("TargetSource cannot determine target class: " +
                "Either an interface or a target is required for proxy creation.");
    }
    // targetClass是接口，直接使用Jdk动态代理
    if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
        return new JdkDynamicAopProxy(config);
    }
    // 使用Cglib
    return new ObjenesisCglibAopProxy(config);
}
else {
    // 使用Jdk动态代理
    return new JdkDynamicAopProxy(config);
}
```

## GetProxy方法,获取代理对象

```java
public Object getProxy() {
		return createAopProxy().getProxy();
	}
```

```java
protected final synchronized AopProxy createAopProxy() {
		if (!this.active) {
			activate();
		}
		return getAopProxyFactory().createAopProxy(this);
	}
```

### DefaultAopProxyFactory

```java
// 如何干扰使用cglib
ProxyFactory factory = new ProxyFactory();
// 设置把接口设置进去
factory.setTargetClass(Interface.class);
factory.addInterface(Interface.class);
factory.setOptimize(false)
```



```java
@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    // config某些属性来自于ProxyFactory的父类
    // 如果想用cglib，可以自己设置成true
    // hasNoUserSuppliedProxyInterfaces 看看有没有添加接口
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
      // 如果设置了一个接口
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
    // jdk动态代理
		else {
			return new JdkDynamicAopProxy(config);
		}
	}
```





### JdkDynamicAopProxy创建代理对象过程

1. 获取生成代理对象所需要实现的接口集合

1. 1. 获取通过ProxyFactory.addInterface()所添加的接口，如果没有通过ProxyFactory.addInterface()添加接口，那么则看ProxyFactory.setTargetClass()所设置的targetClass是不是一个接口，把接口添加到结果集合中
   2. 同时把SpringProxy、Advised、DecoratingProxy这几个接口也添加到结果集合中去

1. 确定好要代理的集合之后，就利用Proxy.newProxyInstance()生成一个代理对象

```java
public class Test {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
		// LubanService lubanService = applicationContext.getBean("lubanService", LubanService.class);

//		lubanService.test();

		ProxyFactory factory = new ProxyFactory();
//		// 目标对象
		factory.setTarget(new LubanService());
		factory.addInterface(LubanInterface.class);
//
		factory.addAdvice(new AfterReturningAdvice(){

			@Override
			public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
				System.out.println(method.getName() + "方法返回后");
			}
		});
//
		factory.addAdvice(new MethodBeforeAdvice() {
			@Override
			public void before(Method method, Object[] args, Object target) throws Throwable {
				System.out.println("执行方法前");
				//method.invoke(target,args);
			}
		});
//
		LubanInterface lubanService = (LubanInterface) factory.getProxy();
//		userService.test();

		lubanService.test();


	}
}
```

```java
public class MethodBeforeAdviceInterceptor implements MethodInterceptor, BeforeAdvice, Serializable {

	private final MethodBeforeAdvice advice;


	/**
	 * Create a new MethodBeforeAdviceInterceptor for the given advice.
	 * @param advice the MethodBeforeAdvice to wrap
	 */
	public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
		Assert.notNull(advice, "Advice must not be null");
		this.advice = advice;
	}


	@Override
  // 这个地方会先调用before方法
	public Object invoke(MethodInvocation mi) throws Throwable {
		this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
		return mi.proceed();
	}

}
```



```java
/**
	 * Implementation of {@code InvocationHandler.invoke}.
	 * <p>Callers will see exactly the exception thrown by the target,
	 * unless a hook method throws an exception.
	 */
	@Override
	@Nullable
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		Object oldProxy = null;
		boolean setProxyContext = false;

		TargetSource targetSource = this.advised.targetSource;
		Object target = null;

		try {
			if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
				// The target does not implement the equals(Object) method itself.
				return equals(args[0]);
			}
			else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
				// The target does not implement the hashCode() method itself.
				return hashCode();
			}
			else if (method.getDeclaringClass() == DecoratingProxy.class) {
				// There is only getDecoratedClass() declared -> dispatch to proxy config.
				return AopProxyUtils.ultimateTargetClass(this.advised);
			}
			else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
					method.getDeclaringClass().isAssignableFrom(Advised.class)) {
				// Service invocations on ProxyConfig with the proxy config...
				return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
			}

			Object retVal;

			if (this.advised.exposeProxy) {
				// Make invocation available if necessary.
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}

			// Get as late as possible to minimize the time we "own" the target,
			// in case it comes from a pool.
			target = targetSource.getTarget();
			Class<?> targetClass = (target != null ? target.getClass() : null);

			// Get the interception chain for this method.
			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

			// Check whether we have any advice. If we don't, we can fallback on direct
			// reflective invocation of the target, and avoid creating a MethodInvocation.
			if (chain.isEmpty()) {
				// We can skip creating a MethodInvocation: just invoke the target directly
				// Note that the final invoker must be an InvokerInterceptor so we know it does
				// nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
				Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
				retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
			}
			else {
				// We need to create a method invocation...
				MethodInvocation invocation =
						new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
				// Proceed to the joinpoint through the interceptor chain.
				retVal = invocation.proceed();
			}

			// Massage return value if necessary.
			Class<?> returnType = method.getReturnType();
			if (retVal != null && retVal == target &&
					returnType != Object.class && returnType.isInstance(proxy) &&
					!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
				// Special case: it returned "this" and the return type of the method
				// is type-compatible. Note that we can't help if the target sets
				// a reference to itself in another returned object.
				retVal = proxy;
			}
			else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
				throw new AopInvocationException(
						"Null return value from advice does not match primitive return type for: " + method);
			}
			return retVal;
		}
		finally {
			if (target != null && !targetSource.isStatic()) {
				// Must have come from TargetSource.
				targetSource.releaseTarget(target);
			}
			if (setProxyContext) {
				// Restore old proxy.
				AopContext.setCurrentProxy(oldProxy);
			}
		}
	}
```



### JdkDynamicAopProxy创建的代理对象执行过程

1. 如果通过ProxyFactory.setExposeProxy()把exposeProxy设置为了true，那么则把代理对象设置到一个ThreadLocal（currentProxy）中去。
2. 获取通过ProxyFactory所设置的target，如果设置的是targetClass，那么target将为null
3. 根据当前所调用的方法对象寻找ProxyFactory中所添加的并匹配的Advisor，并且把Advisor封装为MethodInterceptor返回，得到MethodInterceptor链叫做chain
4. 如果chain为空，则直接执行target对应的当前方法，如果target为null会报错
5. 如果chain不为空，则会依次执行chain中的MethodInterceptor

1. 1. 如果当前MethodInterceptor是MethodBeforeAdviceInterceptor，那么则先执行Advisor中所advice的before()方法，然后执行下一个MethodInterceptor
   2. 如果当前MethodInterceptor是AfterReturningAdviceInterceptor，那么则先执行下一个MethodInterceptor，拿到返回值之后，再执行Advisor中所advice的afterReturning()方法



### ObjenesisCglibAopProxy创建代理对象过程

1. 创建Enhancer
2. 设置Enhancer的superClass为通过ProxyFactory.setTarget()所设置的对象的类
3. 设置Enhancer的interfaces为通过ProxyFactory.addInterface()所添加的接口，以及SpringProxy、Advised接口
4. 设置Enhancer的Callbacks为DynamicAdvisedInterceptor
5. 最后通过Enhancer创建一个代理对象



### ObjenesisCglibAopProxy创建的代理对象执行过程

执行过程主要就看DynamicAdvisedInterceptor中的实现，执行逻辑和JdkDynamicAopProxy中是一样的。







## 使用“自动代理（autoproxy）”功能



"自动代理"表示，只需要在Spring中添加某个Bean，这个Bean是一个BeanPostProcessor，那么Spring在每创建一个Bean时，都会经过这个BeanPostProcessor的判断，去判断当前正在创建的这个Bean是不是需要进行AOP。



我们可以在项目中定义很多个Advisor，定义方式有两种：

1. 通过实现PointcutAdvisor接口
2. 通过@Aspect、@Pointcut、@Before等注解



在创建某个Bean时，会根据当前这个Bean的信息，比如对应的类，以及当前Bean中的方法信息，去和定义的所有Advisor进行匹配，如果匹配到了其中某些Advisor，那么就会把这些Advisor给找出来，并且添加到ProxyFactory中去，在利用ProxyFactory去生成代理对象



### BeanNameAutoProxyCreator



```java
@Bean
public BeanNameAutoProxyCreator creator(){
    BeanNameAutoProxyCreator beanNameAutoProxyCreator = new BeanNameAutoProxyCreator();
    beanNameAutoProxyCreator.setBeanNames("userService");  
    // 代理的配置
    beanNameAutoProxyCreator.setInterceptorNames("myAdvisor");
    return beanNameAutoProxyCreator;
}
```

```java
@Component
public class MyAdvisor implements PointcutAdvisor {

  // 切点再哪一个方法上
	@Override
	public Pointcut getPointcut() {
		NameMatchMethodPointcut nameMatchMethodPointcut = new NameMatchMethodPointcut();
		nameMatchMethodPointcut.addMethodName("test");
		return nameMatchMethodPointcut;
	}

  // 之前执行的方法是什么
	@Override
	public Advice getAdvice() {
		return new MethodBeforeAdvice() {
			@Override
			public void before(Method method, Object[] args, Object target) throws Throwable {
				System.out.println("方法执行前");
			}
		};
	}

	@Override
	public boolean isPerInstance() {
		return false;
	}
}
```



```java
/**
 * Create a proxy with the configured interceptors if the bean is
 * identified as one to proxy by the subclass.
 * @see #getAdvicesAndAdvisorsForBean
 */
@Override
// 第6步所调用的方法
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
   if (bean != null) {
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      // earlyProxyReferences中存的是哪些提前进行了AOP的bean，beanName:AOP之前的对象
      // 注意earlyProxyReferences中并没有存AOP之后的代理对象  BeanPostProcessor
      if (this.earlyProxyReferences.remove(cacheKey) != bean) {
         // 没有提前进行过AOP，则进行AOP
         return wrapIfNecessary(bean, beanName, cacheKey);
      }
   }
   // 为什么不返回代理对象呢？
   return bean;   //
}
```

```java
/**
 * Wrap the given bean if necessary, i.e. if it is eligible for being proxied.
 * @param bean the raw bean instance
 * @param beanName the name of the bean
 * @param cacheKey the cache key for metadata access
 * @return a proxy wrapping the bean, or the raw bean instance as-is
 */
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
   if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
      return bean;
   }
   // 当前这个bean不用被代理
   if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
      return bean;
   }

   // 如果是基础bean，或者@Aspect修饰的类不需要被代理
   if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
      this.advisedBeans.put(cacheKey, Boolean.FALSE);
      return bean;
   }

   // Create proxy if we have advice.
   // 获取当前beanClass所匹配的advisors
   Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);

   // 如果匹配的advisors不等于null，那么则进行代理，并返回代理对象
   if (specificInterceptors != DO_NOT_PROXY) {
      this.advisedBeans.put(cacheKey, Boolean.TRUE);
      // 基于bean对象和Advisor创建代理对象
      Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
      // 存一个代理对象的类型
      this.proxyTypes.put(cacheKey, proxy.getClass());
      return proxy;
   }

   this.advisedBeans.put(cacheKey, Boolean.FALSE);
   return bean;
}
```

## createProxy

```java
/**
	 * Create an AOP proxy for the given bean.
	 * @param beanClass the class of the bean
	 * @param beanName the name of the bean
	 * @param specificInterceptors the set of interceptors that is
	 * specific to this bean (may be empty, but not null)
	 * @param targetSource the TargetSource for the proxy,
	 * already pre-configured to access the bean
	 * @return the AOP proxy for the bean
	 * @see #buildAdvisors
	 */
	protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource) {

		if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
			AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
		}

		ProxyFactory proxyFactory = new ProxyFactory();
		proxyFactory.copyFrom(this);	// 从this复制配置参数
		// 是否指定了必须用cglib进行代理
		if (!proxyFactory.isProxyTargetClass()) {
			// 如果没有指定，那么则判断是不是应该进行cglib代理（判断BeanDefinition中是否指定了要用cglib）
			if (shouldProxyTargetClass(beanClass, beanName)) {
				proxyFactory.setProxyTargetClass(true);
			}
			else {
				// 是否进行jdk动态代理
				evaluateProxyInterfaces(beanClass, proxyFactory); // 判断beanClass有没有实现接口
			}
		}

		// 添加一些commonInterceptors
		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
		proxyFactory.addAdvisors(advisors);	// 向ProxyFactory中添加advisor
		proxyFactory.setTargetSource(targetSource); // 被代理的对象
		customizeProxyFactory(proxyFactory);

		proxyFactory.setFrozen(this.freezeProxy);	//
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}

		// 生成代理对象
		return proxyFactory.getProxy(getProxyClassLoader());
	}
```

## buildAdvisors

```java
protected Advisor[] buildAdvisors(@Nullable String beanName, @Nullable Object[] specificInterceptors) {
		// Handle prototypes correctly...
    // advisor是可以多个类共用的
		Advisor[] commonInterceptors = resolveInterceptorNames();

		List<Object> allInterceptors = new ArrayList<>();
    // 所以这里面除非你指定过advisor才会进入这个条件里面
		if (specificInterceptors != null) {
			allInterceptors.addAll(Arrays.asList(specificInterceptors));
			if (commonInterceptors.length > 0) {
				if (this.applyCommonInterceptorsFirst) {
					allInterceptors.addAll(0, Arrays.asList(commonInterceptors));
				}
				else {
					allInterceptors.addAll(Arrays.asList(commonInterceptors));
				}
			}
		}
		if (logger.isTraceEnabled()) {
			int nrOfCommonInterceptors = commonInterceptors.length;
			int nrOfSpecificInterceptors = (specificInterceptors != null ? specificInterceptors.length : 0);
			logger.trace("Creating implicit proxy for bean '" + beanName + "' with " + nrOfCommonInterceptors +
					" common interceptors and " + nrOfSpecificInterceptors + " specific interceptors");
		}

  	// 如果没有什么判断返回的就是一个空的数组
		Advisor[] advisors = new Advisor[allInterceptors.size()];
		for (int i = 0; i < allInterceptors.size(); i++) {
			advisors[i] = this.advisorAdapterRegistry.wrap(allInterceptors.get(i));
		}
		return advisors;
	}
```



定义的这个bean，相当于一个“自动代理”器，有了这个Bean之后，可以自动的对setBeanNames中所对应的bean进行代理，代理逻辑为所设置的interceptorNames



### **DefaultAdvisorAutoProxyCreator**



**DefaultAdvisorAutoProxyCreator**这个更加强大，只要添加了这个Bean，它就会自动识别所有的Advisor中的PointCut进行代理



![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1604327678682-b514859d-8faf-4a39-a7cc-968c627af901.png)





**AbstractAutoProxyCreator**实现了SmartInstantiationAwareBeanPostProcessor接口，是一个BeanPostProcessor

1. 在某个Bean**实例化之前**，查看该AbstractAutoProxyCreator中是不是设置了CustomTargetSource，如果设置了就查看当前Bean是不是需要创建一个TargetSource，如果需要就会创建一个TargetSource对象，然后进行AOP创建一个代理对象，并返回该代理对象
2. 如果某个Bean出现了循环依赖，那么会利用getEarlyBeanReference()方法提前进行AOP
3. 在某个Bean**初始化之后**，会调用wrapIfNecessary()方法进行AOP
4. 在这个类中提供了一个抽象方法：getAdvicesAndAdvisorsForBean()，表示对于某个Bean匹配了哪些Advices和Advisors



**AbstractAdvisorAutoProxyCreator**继承了AbstractAutoProxyCreator，AbstractAdvisorAutoProxyCreator中实现了getAdvicesAndAdvisorsForBean()方法，实现逻辑为：

1. 调用findEligibleAdvisors()

1. 1. 调用findCandidateAdvisors，得到所有Advisor类型的Bean
   2. 按当前正在进行Bean的生命周期的Bean进行过滤



## @EnableAspectJAutoProxy

这个注解主要是添加了一个AnnotationAwareAspectJAutoProxyCreator类型的BeanDefinition

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1600784104850-e542baca-7a29-4396-9223-1dfc36418bc8.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)



**AspectJAwareAdvisorAutoProxyCreator**继承了AbstractAdvisorAutoProxyCreator，重写了shouldSkip(Class<?> beanClass, String beanName)方法，表示某个bean需不需要进行AOP，在shouldSkip()方法中：

1. 拿到所有的Advisor
2. 遍历所有的Advisor，如果当前bean是AspectJPointcutAdvisor，那么则跳过



**AnnotationAwareAspectJAutoProxyCreator**继承了AspectJAwareAdvisorAutoProxyCreator，重写了findCandidateAdvisors()方法，它即可以找到Advisor类型的bean，也能把所有@Aspect注解标注的类扫描出来并生成Advisor



## 注解和源码对应关系

1. @Before对应的是AspectJMethodBeforeAdvice，直接实现MethodBeforeAdvice，在进行动态代理时会把AspectJMethodBeforeAdvice转成MethodBeforeAdviceInterceptor，也就转变成了MethodBeforeAdviceInterceptor

1. 1. 先执行advice对应的方法
   2. 再执行MethodInvocation的proceed()，会执行下一个Interceptor，如果没有下一个Interceptor了，会执行target对应的方法

1. @After对应的是AspectJAfterAdvice，直接实现了MethodInterceptor

1. 1. 先执行MethodInvocation的proceed()，会执行下一个Interceptor，如果没有下一个Interceptor了，会执行target对应的方法
   2. 再执行advice对应的方法

1. @Around对应的是AspectJAroundAdvice，直接实现了MethodInterceptor

1. 1. 直接执行advice对应的方法

1. @AfterThrowing对应的是AspectJAfterThrowingAdvice，直接实现了MethodInterceptor

1. 1. 先执行MethodInvocation的proceed()，会执行下一个Interceptor，如果没有下一个Interceptor了，会执行target对应的方法
   2. 如果上面抛了Throwable，那么则会执行advice对应的方法

1. @AfterReturning对应的是AspectJAfterReturningAdvice，实现了AfterReturningAdvice，在进行动态代理时会把AspectJAfterReturningAdvice转成AfterReturningAdviceInterceptor，也就转变成了MethodInterceptor

1. 1. 先执行MethodInvocation的proceed()，会执行下一个Interceptor，如果没有下一个Interceptor了，会执行target对应的方法
   2. 执行上面的方法后得到最终的方法的返回值
   3. 再执行Advice对应的方法