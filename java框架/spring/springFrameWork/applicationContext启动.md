



## BeanFactoryPostProcessor

叫做BeanFactory的后置处理器，和Bean的后置处理器对比理解。

BeanPostProcessor是用来对Bean进行处理的，

BeanFactoryPostProcessor是用来对BeanFactory进行处理的。





## 如何理解refresh()？



```
/**
     * Load or refresh the persistent representation of the configuration,
     * which might an XML file, properties file, or relational database schema.
     * <p>As this is a startup method, it should destroy already created singletons
     * if it fails, to avoid dangling resources. In other words, after invocation
     * of that method, either all or no singletons at all should be instantiated.
     * @throws BeansException if the bean factory could not be initialized
     * @throws IllegalStateException if already initialized and multiple refresh
     * attempts are not supported
     */
    void refresh() throws BeansException, IllegalStateException;
```



这是ConfigurableApplicationContext接口上refresh()方法的注释，意思是：加载或刷新持久化的配置，可能是XML文件、属性文件或关系数据库中存储的。由于这是一个启动方法，如果失败，它应该销毁已经创建的单例，以避免暂用资源。换句话说，在调用该方法之后，应该实例化所有的单例，或者根本不实例化单例 。



有个理念需要注意：**ApplicationContext关闭之后不代表JVM也关闭了，ApplicationContext是属于JVM的，说白了ApplicationContext也是JVM中的一个对象。**

**
**

在Spring的设计中，也提供可以刷新的ApplicationContext和不可以刷新的ApplicationContext。比如：

```
AbstractRefreshableApplicationContext extends AbstractApplicationContext
```

就是可以刷新的



```
GenericApplicationContext extends AbstractApplicationContext
```

就是不可以刷新的。



AnnotationConfigApplicationContext继承的是GenericApplicationContext，所以它是不能刷新的。

AnnotationConfigWebApplicationContext继承的是AbstractRefreshableWebApplicationContext，所以它是可以刷的。



上面说的**不能刷新是指不能重复刷新，只能调用一次refresh方法，第二次时会报错。**



## refresh做了哪些事情？



![ApplicationContext启动流程详解 (1).png](https://cdn.nlark.com/yuque/0/2020/png/365147/1603458878384-545be0bc-b56c-4dae-b5a6-df19e9dc2f1b.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)





下面以AnnotationConfigApplicationContext为例子，来介绍refresh的底层原理。



1. 在调用AnnotationConfigApplicationContext的构造方法之前，会调用父类GenericApplicationContext的无参构造方法，会构造一个BeanFactory，为**DefaultListableBeanFactory**。
2. 构造AnnotatedBeanDefinitionReader（**主要作用添加一些基础的PostProcessor，同时可以通过reader进行BeanDefinition的注册**），同时对BeanFactory进行设置和添加**PostProcessor**（后置处理器）

1. 1. 设置dependencyComparator：AnnotationAwareOrderComparator，它是一个Comparator，是用来进行排序的，会获取某个对象上的**Order注解**或者通过实现**Ordered接口**所定义的值进行排序，在日常开发中可以利用这个类来进行排序。
   2. 设置autowireCandidateResolver：ContextAnnotationAutowireCandidateResolver，用来解析某个Bean能不能进行自动注入，比如某个Bean的autowireCandidate属性是否等于true
   3. 向BeanFactory中添加**ConfigurationClassPostProcessor**对应的BeanDefinition，它是一个BeanDefinitionRegistryPostProcessor，并且实现了PriorityOrdered接口
   4. 向BeanFactory中添加**AutowiredAnnotationBeanPostProcessor**对应的BeanDefinition，它是一个InstantiationAwareBeanPostProcessorAdapter，MergedBeanDefinitionPostProcessor
   5. 向BeanFactory中添加CommonAnnotationBeanPostProcessor对应的BeanDefinition，它是一个InstantiationAwareBeanPostProcessor，InitDestroyAnnotationBeanPostProcessor
   6. 向BeanFactory中添加EventListenerMethodProcessor对应的BeanDefinition，它是一个BeanFactoryPostProcessor，SmartInitializingSingleton
   7. 向BeanFactory中添加DefaultEventListenerFactory对应的BeanDefinition，它是一个EventListenerFactory

1. 构造ClassPathBeanDefinitionScanner（**主要作用可以用来扫描得到并注册BeanDefinition**），同时进行设置：

1. 1. 设置**this.includeFilters = AnnotationTypeFilter(****Component****.class)**
   2. 设置environment
   3. 设置resourceLoader

1. 利用reader注册AppConfig为BeanDefinition，类型为AnnotatedGenericBeanDefinition
2. **接下来就是调用refresh方法**
3. prepareRefresh()：

1. 1. 记录启动时间
   2. 可以允许子容器设置一些内容到Environment中
   3. 验证Environment中是否包括了必须要有的属性

1. obtainFreshBeanFactory()：进行BeanFactory的refresh，在这里会去调用子类的refreshBeanFactory方法，具体子类是怎么刷新的得看子类，然后再调用子类的getBeanFactory方法，重新得到一个BeanFactory
2. prepareBeanFactory(beanFactory)：

1. 1. 设置beanFactory的类加载器
   2. 设置表达式解析器：StandardBeanExpressionResolver，用来解析Spring中的表达式
   3. 添加PropertyEditorRegistrar：ResourceEditorRegistrar，PropertyEditor类型转化器注册器，用来注册一些默认的PropertyEditor
   4. 添加一个Bean的后置处理器：ApplicationContextAwareProcessor，是一个BeanPostProcessor，用来执行EnvironmentAware、ApplicationEventPublisherAware等回调方法
   5. 添加**ignoredDependencyInterface**：可以向这个属性中添加一些接口，如果某个类实现了这个接口，并且这个类中的某些set方法在接口中也存在，那么这个set方法在自动注入的时候是不会执行的，比如EnvironmentAware这个接口，如果某个类实现了这个接口，那么就必须实现它的setEnvironment方法，而这是一个set方法，和Spring中的autowire是冲突的，那么Spring在自动注入时是不会调用setEnvironment方法的，而是等到回调Aware接口时再来调用（注意，这个功能仅限于xml的autowire，@Autowired注解是忽略这个属性的）

1. 1. 1. EnvironmentAware
      2. EmbeddedValueResolverAware
      3. ResourceLoaderAware
      4. ApplicationEventPublisherAware
      5. MessageSourceAware
      6. ApplicationContextAware
      7. 另外其实在构造BeanFactory的时候就已经提前添加了另外三个：
      8. BeanNameAware
      9. BeanClassLoaderAware
      10. BeanFactoryAware

1. 1. 添加**resolvableDependencies**：在byType进行依赖注入时，会先从这个属性中根据类型找bean

1. 1. 1. BeanFactory.class：当前BeanFactory对象
      2. ResourceLoader.class：当前ApplicationContext对象
      3. ApplicationEventPublisher.class：当前ApplicationContext对象
      4. ApplicationContext.class：当前ApplicationContext对象

1. 1. 添加一个Bean的后置处理器：ApplicationListenerDetector，是一个BeanPostProcessor，用来判断某个Bean是不是ApplicationListener，如果是则把这个Bean添加到ApplicationContext中去，注意一个ApplicationListener只能是单例的
   2. 添加一个Bean的后置处理器：LoadTimeWeaverAwareProcessor，是一个BeanPostProcessor，用来判断某个Bean是不是实现了LoadTimeWeaverAware接口，如果实现了则把ApplicationContext中的loadTimeWeaver回调setLoadTimeWeaver方法设置给该Bean。
   3. 添加一些单例bean到单例池：

1. 1. 1. "environment"：Environment对象
      2. "systemProperties"：System.getProperties()返回的Map对象
      3. "systemEnvironment"：System.getenv()返回的Map对象

1. postProcessBeanFactory(beanFactory) ： 提供给AbstractApplicationContext的子类进行扩展，具体的子类，可以继续向BeanFactory中再添加一些东西
2. invokeBeanFactoryPostProcessors(beanFactory)：**执行BeanFactoryPostProcessor**

1. 1. 此时在BeanFactory中会存在一个BeanFactoryPostProcessor：**ConfigurationClassPostProcessor**，它也是一个**BeanDefinitionRegistryPostProcessor**
   2. **第一阶段**
   3. 从BeanFactory中找到类型为BeanDefinitionRegistryPostProcessor的beanName，也就是**ConfigurationClassPostProcessor**， 然后调用BeanFactory的getBean方法得到实例对象
   4. 执行**ConfigurationClassPostProcessor的****postProcessBeanDefinitionRegistry()**方法:

1. 1. 1. 解析AppConfig类
      2. 扫描得到BeanDefinition并注册
      3. 解析@Import，@Bean等注解得到BeanDefinition并注册
      4. 详细的看另外的笔记，专门分析了**ConfigurationClassPostProcessor是如何工作的**
      5. 在这里，我们只需要知道在这一步会去得到BeanDefinition，而这些BeanDefinition中可能存在BeanFactoryPostProcessor和BeanDefinitionRegistryPostProcessor，所以执行完ConfigurationClassPostProcessor的postProcessBeanDefinitionRegistry()方法后，还需要继续执行其他BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry()方法

1. 1. 执行其他BeanDefinitionRegistryPostProcessor的**postProcessBeanDefinitionRegistry()**方法
   2. 执行所有BeanDefinitionRegistryPostProcessor的**postProcessBeanFactory()**方法
   3. **第二阶段**
   4. 从BeanFactory中找到类型为BeanFactoryPostProcessor的beanName，而这些BeanFactoryPostProcessor包括了上面的BeanDefinitionRegistryPostProcessor
   5. 执行还没有执行过的BeanFactoryPostProcessor的**postProcessBeanFactory()**方法

1. 到此，所有的BeanFactoryPostProcessor的逻辑都执行完了，主要做的事情就是得到BeanDefinition并注册到BeanFactory中
2. registerBeanPostProcessors(beanFactory)：因为上面的步骤完成了扫描，这个过程中程序员可能自己定义了一些BeanPostProcessor，在这一步就会把BeanFactory中所有的BeanPostProcessor找出来并实例化得到一个对象，并添加到BeanFactory中去（属性**beanPostProcessors**），最后再重新添加一个ApplicationListenerDetector对象（之前其实就添加了过，这里是为了把ApplicationListenerDetector移动到最后）
3. initMessageSource()：如果BeanFactory中存在一个叫做"**messageSource**"的BeanDefinition，那么就会把这个Bean对象创建出来并赋值给ApplicationContext的messageSource属性，让ApplicationContext拥有**国际化**的功能
4. initApplicationEventMulticaster()：如果BeanFactory中存在一个叫做"**applicationEventMulticaster**"的BeanDefinition，那么就会把这个Bean对象创建出来并赋值给ApplicationContext的applicationEventMulticaster属性，让ApplicationContext拥有**事件发布**的功能
5. onRefresh()：提供给AbstractApplicationContext的子类进行扩展，没用
6. registerListeners()：从BeanFactory中获取ApplicationListener类型的beanName，然后添加到ApplicationContext中的事件广播器**applicationEventMulticaster**中去，到这一步因为FactoryBean还没有调用getObject()方法生成Bean对象，所以这里要在根据类型找一下ApplicationListener，记录一下对应的beanName
7. finishBeanFactoryInitialization(beanFactory)：完成BeanFactory的初始化，主要就是**实例化非懒加载的单例Bean**，单独的笔记去讲。
8. finishRefresh()：BeanFactory的初始化完后，就到了Spring启动的最后一步了

1. 1. 设置ApplicationContext的lifecycleProcessor，默认情况下设置的是DefaultLifecycleProcessor
   2. 调用lifecycleProcessor的onRefresh()方法，如果是DefaultLifecycleProcessor，那么会获取所有类型为Lifecycle的Bean对象，然后调用它的start()方法，这就是ApplicationContext的生命周期扩展机制
   3. 发布**ContextRefreshedEvent**事件



spring启动的目的--> 获取bean

0.生成beandefinition

1.类加载器。

2.@value 填充 占位符填充对象，解析springEL表达式

3.AutowiredAnnotationBeanPostProcessor 后置处理器

```java
//扫描配置类，解析配置类
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

### 启动spring过程

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		// 初始化AnnotatedBeanDefinitionReader 和 ClassPathBeanDefinitionScanner
		// 实际上reader和scanner二者用其一就好了，两者都是在Spring启动之处注册BeanDefinition
		// 比如当前这个类，调用register方法实际上就是利用的reader去注册一个componentClasses对应的beanDefinition
		// 而我们可以另外一个构造方法AnnotationConfigApplicationContext(String... basePackages),这个构造方法里使用的是scanner去扫描包路径得到BeanDefinition
		this();
		// 把AppConfig类注册为一个AnnotatedGenericBeanDefinition到reader中
		register(componentClasses);

		// 在Spring启动到这里时，Spring中已经存在了一些BeanDefinition了，注意，不是Bean
		// 刷新我们可以理解为，去解析这个BeanDefinition，因为这里的每个BeanDefinition都代表不同的意义，所做的事情也不同
		refresh();
	}
```



1. 调用父类的无参构造（生成bean工厂）

   ```java
   /**
   	 * Create a new GenericApplicationContext.
   	 * @see #registerBeanDefinition
   	 * @see #refresh
   	 */
   	public GenericApplicationContext() {
   		this.beanFactory = new DefaultListableBeanFactory();
   	}
   ```

   

2. 再调用自己的无参构造

   ```java
   public AnnotationConfigApplicationContext() {
   		// 再执行这个构造方法之前，会先执行父类的构造方法，会初始化一个beanFactory = new DefaultListableBeanFactory()
   
   		// 生成5个原始的beanDefinition
   		// 后面会生成对应的bean，然后做特定的事情
       //可以扫描单个类
   		this.reader = new AnnotatedBeanDefinitionReader(this);
   		// AnnotatedBeanDefinitionReader负责将某个类注册为beanDefinition
   		// reader.register(AppConfig.class);
   		//扫描某个路径下面类
   		this.scanner = new ClassPathBeanDefinitionScanner(this);
   	}
   ```

   ## AnnotatedBeanDefinitionReader

   ```java
   public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
   		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
   		Assert.notNull(environment, "Environment must not be null");
   		this.registry = registry;
   		this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
   
   		// 构造Reader时就注册了一些Processor
   		// 1.ConfigurationClassPostProcessor
   		// 2.AutowiredAnnotationBeanPostProcessor
   		// 3.CommonAnnotationBeanPostProcessor
   		// 4.EventListenerMethodProcessor
   		// 5.DefaultEventListenerFactory
   		AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
   	}
   ```

   

   ```java
   public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
   			BeanDefinitionRegistry registry, @Nullable Object source) {
   
   		DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
   		if (beanFactory != null) {
   			// 设置beanFactory的OrderComparator，为AnnotationAwareOrderComparator
   			// beanFactory利用OrderComparator对bean进行排序，通过Ordered接口，Ordered注解，Priority注解指定排序
   			if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
   				beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
   			}
   			// 设置自动注入候选者的解析器
         //判断autowired的时候，解析属性上有没有lazy注解
   			if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
   				beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
   			}
   		}
   
   		Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);
   
   		// 注册ConfigurationClassPostProcessor类型的BeanDefinition
     	//配置类的处理器
   		if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
   			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
   			def.setSource(source);
   			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
   		}
   
   		// 注册AutowiredAnnotationBeanPostProcessor类型的BeanDefinition
   		if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
   			RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
   			def.setSource(source);
   			beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
   		}
   
   		// 注册CommonAnnotationBeanPostProcessor类型的BeanDefinition
   		// Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
   		if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
   			RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
   			def.setSource(source);
   			beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
   		}
   
   		// 注册PersistenceAnnotationBeanPostProcessor类型的BeanDefinition
   		// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
   		if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
   			RootBeanDefinition def = new RootBeanDefinition();
   			try {
   				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
   						AnnotationConfigUtils.class.getClassLoader()));
   			}
   			catch (ClassNotFoundException ex) {
   				throw new IllegalStateException(
   						"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
   			}
   			def.setSource(source);
   			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
   		}
   
   		// 注册EventListenerMethodProcessor类型的BeanDefinition
   		if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
   			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
   			def.setSource(source);
   			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
   		}
   
   		// 注册DefaultEventListenerFactory类型的BeanDefinition
   		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
   			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
   			def.setSource(source);
   			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
   		}
   
   		return beanDefs;
   	}
   ```

   ## ClassPathBeanDefinitionScanner

   ```java
   /**
   	 * Create a new {@code ClassPathBeanDefinitionScanner} for the given bean factory and
   	 * using the given {@link Environment} when evaluating bean definition profile metadata.
   	 * @param registry the {@code BeanFactory} to load bean definitions into, in the form
   	 * of a {@code BeanDefinitionRegistry}
   	 * @param useDefaultFilters whether to include the default filters for the
   	 * {@link org.springframework.stereotype.Component @Component},
   	 * {@link org.springframework.stereotype.Repository @Repository},
   	 * {@link org.springframework.stereotype.Service @Service}, and
   	 * {@link org.springframework.stereotype.Controller @Controller} stereotype annotations
   	 * @param environment the Spring {@link Environment} to use when evaluating bean
   	 * definition profile metadata
   	 * @param resourceLoader the {@link ResourceLoader} to use
   	 * @since 4.3.6
   	 */
   	public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
   			Environment environment, @Nullable ResourceLoader resourceLoader) {
   
   		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
   		this.registry = registry;
   
   		// 添加includeFilters
   		if (useDefaultFilters) {
   			registerDefaultFilters();
   		}
   		setEnvironment(environment);
   		setResourceLoader(resourceLoader);
   	}
   
   ```

   ```java
   /**
   	 * Register the default filter for {@link Component @Component}.
   	 * <p>This will implicitly register all annotations that have the
   	 * {@link Component @Component} meta-annotation including the
   	 * {@link Repository @Repository}, {@link Service @Service}, and
   	 * {@link Controller @Controller} stereotype annotations.
   	 * <p>Also supports Java EE 6's {@link javax.annotation.ManagedBean} and
   	 * JSR-330's {@link javax.inject.Named} annotations, if available.
   	 * scanner里面有筛选条件，会解析带有@Component注解的类
   	 */
   protected void registerDefaultFilters() {
   		this.includeFilters.add(new AnnotationTypeFilter(Component.class));
   		ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
   		try {
   			this.includeFilters.add(new AnnotationTypeFilter(
   					((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
   			logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
   		}
   		catch (ClassNotFoundException ex) {
   			// JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
   		}
   		try {
   			this.includeFilters.add(new AnnotationTypeFilter(
   					((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
   			logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
   		}
   		catch (ClassNotFoundException ex) {
   			// JSR-330 API not available - simply skip.
   		}
   	}
   ```

   

   ## 以上走完this初始化了一些类之后，开始走register方法

   ```java
   public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
   		// 初始化AnnotatedBeanDefinitionReader 和 ClassPathBeanDefinitionScanner
   		// 实际上reader和scanner二者用其一就好了，两者都是在Spring启动之处注册BeanDefinition
   		// 比如当前这个类，调用register方法实际上就是利用的reader去注册一个componentClasses对应的beanDefinition
   		// 而我们可以另外一个构造方法AnnotationConfigApplicationContext(String... basePackages),这个构造方法里使用的是scanner去扫描包路径得到BeanDefinition
   		this();
   		// 把AppConfig类注册为一个AnnotatedGenericBeanDefinition到reader中
   		register(componentClasses);
   
   		// 在Spring启动到这里时，Spring中已经存在了一些BeanDefinition了，注意，不是Bean
   		// 刷新我们可以理解为，去解析这个BeanDefinition，因为这里的每个BeanDefinition都代表不同的意义，所做的事情也不同
   		refresh();
   	}
   ```

   ## doRegisterBean把我自己传进来的配置类注册成为一个bean

   ```java
   /**
   	 * Register a bean from the given bean class, deriving its metadata from
   	 * class-declared annotations.
   	 * @param beanClass the class of the bean
   	 * @param name an explicit name for the bean
   	 * @param qualifiers specific qualifier annotations to consider, if any,
   	 * in addition to qualifiers at the bean class level
   	 * @param supplier a callback for creating an instance of the bean
   	 * (may be {@code null})
   	 * @param customizers one or more callbacks for customizing the factory's
   	 * {@link BeanDefinition}, e.g. setting a lazy-init or primary flag
   	 * @since 5.0
   	 *
   	 * 这个方法是注册一个bean最底层的方法，参数很多
   	 * beanClass表示bean的类型
   	 * name表示bean的名字
   	 * qualifiers表示资格限定器，如果某个类上没有写@Lazy注解，但是在调用registerBean方法时传递了Lazy.class，那么则达到了一样的效果
   	 * supplier表示实例提供器，如果指定了supplier，那么bean的实例是有这个supplier生成的
   	 * customizers表示BeanDefinition自定义器，可以通过customizers对BeanDefinition进行自定义修改
   	 */
   	private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
   			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
   			@Nullable BeanDefinitionCustomizer[] customizers) {
   
   		// 直接生成一个AnnotatedGenericBeanDefinition
   		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
   		// 判断当前abd是否被标注了@Conditional注解，并判断是否符合所指定的条件，如果不符合，则跳过，不进行注册
   		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
   			return;
   		}
   
   		// 设置supplier、scope属性，以及得到beanName
   		abd.setInstanceSupplier(supplier);
   		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
   		abd.setScope(scopeMetadata.getScopeName());
   		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
   
   		// 获取Lazy、Primary、DependsOn、Role、Description注解信息并设置给abd
   		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
   		if (qualifiers != null) {
   			for (Class<? extends Annotation> qualifier : qualifiers) {
   				if (Primary.class == qualifier) {
   					abd.setPrimary(true);
   				}
   				else if (Lazy.class == qualifier) {
   					abd.setLazyInit(true);
   				}
   				else {
   					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
   				}
   			}
   		}
   		// 使用自定义器修改BeanDefinition
   		if (customizers != null) {
   			for (BeanDefinitionCustomizer customizer : customizers) {
   				customizer.customize(abd);
   			}
   		}
   
   		// BeanDefinition中是没有beanName的，BeanDefinitionHolder中持有了BeanDefinition,beanName,alias
   		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
   		// 解析Scope中的ProxyMode属性，默认为no，不生成代理对象
   		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
   
   		// 注册到registry中
   		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
   	}
   ```

   

   

   

   

3. 调用refresh（）

   ```java
   public void refresh() throws BeansException, IllegalStateException {
   		synchronized (this.startupShutdownMonitor) {
   			// Prepare this context for refreshing.
   			prepareRefresh();
   
   			// Tell the subclass to refresh the internal bean factory.
   			// 获取初始化好了的beanFactory
   			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   
   			// Prepare the bean factory for use in this context.
   			// 初始化BeanFactory
   			// 比如向BeanFactory添加了两个BeanPostProcessor
   			// 设置ignoreDependencyInterface
   			// 把BeanFactory对象和ApplicationContext对象添加到Spring容器中
   			prepareBeanFactory(beanFactory);
   
   			try {
   				// Allows post-processing of the bean factory in context subclasses.
   				// 子类可以对BeanFactory进行进一步初始化
   				postProcessBeanFactory(beanFactory);
   
   				// Invoke factory processors registered as beans in the context.
   				// BeanFactory初始化了之后，执行BeanFactoryPostProcessors，进一步处理BeanFactory
   				// 在这一步中会执行ConfigurationClassPostProcessor进行@Component的扫描，扫描得到BeanDefinition，并注册到beanFactory中
   				invokeBeanFactoryPostProcessors(beanFactory);
   
   				// Register bean processors that intercept bean creation.
   				// 注册可以拦截bean的创建的BeanPostProcessor，
   				registerBeanPostProcessors(beanFactory);
   
   				// Initialize message source for this context.
   				// 初始化MessageSource，如果配置了一个名字叫做“messageSource”的BeanDefinition
   				// 就会把这个Bean创建出来，并赋值给ApplicationContext的messageSource属性
   				// 这样ApplicationContext就可以使用国际化的功能了
   				initMessageSource();
   
   				// Initialize event multicaster for this context.
   				// 设置ApplicationContext的applicationEventMulticaster
   				initApplicationEventMulticaster();
   
   				// Initialize other special beans in specific context subclasses.
   				// 执行子类的onRefresh方法
   				onRefresh();
   
   				// Check for listener beans and register them.
   				// 注册Listener
   				registerListeners();
   
   				// Instantiate all remaining (non-lazy-init) singletons.
   				// 完成beanFactory的初始化（实例化非懒加载的单例bean）
   				finishBeanFactoryInitialization(beanFactory);
   
   				// Last step: publish corresponding event.
   				// 发布事件
   				finishRefresh();
   			}
   
   			catch (BeansException ex) {
   				if (logger.isWarnEnabled()) {
   					logger.warn("Exception encountered during context initialization - " +
   							"cancelling refresh attempt: " + ex);
   				}
   
   				// Destroy already created singletons to avoid dangling resources.
   				destroyBeans();
   
   				// Reset 'active' flag.
   				cancelRefresh(ex);
   
   				// Propagate exception to caller.
   				throw ex;
   			}
   
   			finally {
   				// Reset common introspection caches in Spring's core, since we
   				// might not ever need metadata for singleton beans anymore...
   				resetCommonCaches();
   			}
   		}
   	}
   ```

   

   ## prepareRefesh()

   ```java
   protected void prepareRefresh() {
   		// Switch to active.
   		this.startupDate = System.currentTimeMillis();
   		this.closed.set(false);
   		this.active.set(true);
   
   		if (logger.isDebugEnabled()) {
   			if (logger.isTraceEnabled()) {
   				logger.trace("Refreshing " + this);
   			}
   			else {
   				logger.debug("Refreshing " + getDisplayName());
   			}
   		}
   
   		// Initialize any placeholder property sources in the context environment.
       //其中的init方法Annotation里面是没有的，交给不同的子类去实现，这个如果用Generic里面有
       //会setEnvironment
     	//@PropertySouce注解定义的类,里面的信息也会存在环境里面
       //servlet会用
   		initPropertySources();
   
   		// Validate that all properties marked as required are resolvable:
   		// see ConfigurablePropertyResolver#setRequiredProperties
     	// applicationContext.getEnvironment().setResource("sfsdfsd")可以这样设置
     	// 检查Environment里面一定要存在的key是否存在
   		getEnvironment().validateRequiredProperties();
   
   		// Store pre-refresh ApplicationListeners...
   		if (this.earlyApplicationListeners == null) {
   			this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
   		}
   		else {
   			// Reset local application listeners to pre-refresh state.
   			this.applicationListeners.clear();
   			this.applicationListeners.addAll(this.earlyApplicationListeners);
   		}
   
   		// Allow for the collection of early ApplicationEvents,
   		// to be published once the multicaster is available...
   		this.earlyApplicationEvents = new LinkedHashSet<>();
   	}
   ```

   

   ## obtainFreshBeanFactory

   ```java
   /**
   	 * Tell the subclass to refresh the internal bean factory.
   	 * @return the fresh BeanFactory instance
   	 * @see #refreshBeanFactory()
   	 * @see #getBeanFactory()
   	 */
   	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
   		refreshBeanFactory();
       //如果支持刷新，拿的就是刷新之后重新赋值的beanfacotory
       //如果不支持刷新，那么就是那启动的时候在构造方法里面赋值的beanfactory，DefaultListableBeanFactory
   		return getBeanFactory();
   	}
   ```

   ```java
   /**
   	 * Do nothing: We hold a single internal BeanFactory and rely on callers
   	 * to register beans through our public methods (or the BeanFactory's).
   	 * @see #registerBeanDefinition
   	 * Generic这个类不能重复刷新，如果换成别的applicationContext
   	 比如AnnotationWebApplicationContext他就可以刷新
   	 */
   	@Override
   	protected final void refreshBeanFactory() throws IllegalStateException {
   		if (!this.refreshed.compareAndSet(false, true)) {
   			throw new IllegalStateException(
   					"GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
   		}
   		this.beanFactory.setSerializationId(getId());
   	}
   ```

   ## prepareBeanFactory()方法

   ```java
   /**
   	 * Configure the factory's standard context characteristics,
   	 * such as the context's ClassLoader and post-processors.
   	 * @param beanFactory the BeanFactory to configure
   	 */
   	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
   		// Tell the internal bean factory to use the context's class loader etc.
   		beanFactory.setBeanClassLoader(getClassLoader());
       // el表达式的解析器
   		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
       //设置默认的类型转化器
   		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
   
   		// Configure the bean factory with context callbacks.
       //这个后置处理器会判断你一个bean上是不是实现了Aware这种回调接口
       //回调的时候也是他去执行的
   		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
   
   		// 如果一个属性对应的set方法在ignoredDependencyInterfaces接口中被定义了，则该属性不会进行自动注入（是Spring中的自动注入，不是@Autowired）
       //如果用的xml里面的bytype,byname依赖注入的时候就会走这个逻辑，查询出他是不是实现于spring内置的接口
       //但是@Autowired流程跟单纯的byname，bytype是不一样的，所以@Autowired不会过滤
   		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
   		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
   		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
   		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
   		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
   		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
   
   		// BeanFactory interface not registered as resolvable type in a plain factory.
   		// MessageSource registered (and found for autowiring) as a bean.
   		// 相当于直接把ApplicationContext对象放入Bean工厂中
   		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
   		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
   		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
   		beanFactory.registerResolvableDependency(ApplicationContext.class, this);
   
   		// Register early post-processor for detecting inner beans as ApplicationListeners.
       //
   		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
   
   		// Detect a LoadTimeWeaver and prepare for weaving, if found.
   		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
   			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
   			// Set a temporary ClassLoader for type matching.
   			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
   		}
   
   		// Register default environment beans.
   		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
   			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
   		}
       // 操作系统
   		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
   			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
   		}
       // jvm
   		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
   			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
   		}
   	}
   ```

   

   ## postProcessBeanFactory()

   ```java
   /**
   	 * Modify the application context's internal bean factory after its standard
   	 * initialization. All bean definitions will have been loaded, but no beans
   	 * will have been instantiated yet. This allows for registering special
   	 * BeanPostProcessors etc in certain ApplicationContext implementations.
   	 * @param beanFactory the bean factory used by the application context
   	 如果子类想要再去实现bean后置处理器，这里自己实现,比如说servlet
   	 */
   	protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
   	}
   ```

   ### 主要是不通的字类这个方法注册的东西也不一样

   ```java
   /**
   	 * Register ServletContextAwareProcessor.
   	 * @see ServletContextAwareProcessor
   	 */
   	@Override
   	protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
   		if (this.servletContext != null) {
   			beanFactory.addBeanPostProcessor(new ServletContextAwareProcessor(this.servletContext));
   			beanFactory.ignoreDependencyInterface(ServletContextAware.class);
   		}
   		WebApplicationContextUtils.registerWebApplicationScopes(beanFactory, this.servletContext);
   		WebApplicationContextUtils.registerEnvironmentBeans(beanFactory, this.servletContext);
   	}
   ```

   

   ## invokeBeanFactoryPostProcessors

   ```java
   /**
   	 * Instantiate and invoke all registered BeanFactoryPostProcessor beans,
   	 * respecting explicit order if given.
   	 * <p>Must be called before singleton instantiation.
   	 */
   	protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
   		PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
   
   		// Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
   		// (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
   		if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
   			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
   			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
   		}
   	}
   ```

   

   ## registerBeanPostProcessors()

   ```java
   /**
   	 * Instantiate and register all BeanPostProcessor beans,
   	 * respecting explicit order if given.
   	 * <p>Must be called before any instantiation of application beans.
   	 */
   	protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
   		PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
   	}
   ```

   ```java
   public static void registerBeanPostProcessors(
   			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
   
   		// 获取BeanPostProcessor类型的bean
   		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
   
   		// Register BeanPostProcessorChecker that logs an info message when
   		// a bean is created during BeanPostProcessor instantiation, i.e. when
   		// a bean is not eligible for getting processed by all BeanPostProcessors.
       // 启动初始化的时候可能已经有一些后置处理器了
   		int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
   		beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));
   
   		// Separate between BeanPostProcessors that implement PriorityOrdered,
   		// Ordered, and the rest.
   		List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
   		List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
   		List<String> orderedPostProcessorNames = new ArrayList<>();
   		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
   		for (String ppName : postProcessorNames) {
         //判断是不是实现PriorityOrder这个接口
         //Order只表示顺序
         //Priority有级别划分
         //实现这个接口就相当于优先级最高
   			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
   				BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
   				priorityOrderedPostProcessors.add(pp);
   				if (pp instanceof MergedBeanDefinitionPostProcessor) {
   					internalPostProcessors.add(pp);
   				}
   			}
         //这个是第二个等级
   			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
   				orderedPostProcessorNames.add(ppName);
   			}
         //全都没实现，那就是第三等级
   			else {
   				nonOrderedPostProcessorNames.add(ppName);
   			}
   		}
   
     //按照刚才的排序，把刚才的放在bean工厂里面
   		// First, register the BeanPostProcessors that implement PriorityOrdered.
   		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
   		// 把priorityOrderedPostProcessors添加到beanFactory中
   		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);
   
   		// Next, register the BeanPostProcessors that implement Ordered.
   		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
   		for (String ppName : orderedPostProcessorNames) {
   			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
   			orderedPostProcessors.add(pp);
   			if (pp instanceof MergedBeanDefinitionPostProcessor) {
   				internalPostProcessors.add(pp);
   			}
   		}
   		sortPostProcessors(orderedPostProcessors, beanFactory);
   		// 把orderedPostProcessors添加到beanFactory中
   		registerBeanPostProcessors(beanFactory, orderedPostProcessors);
   
   		// Now, register all regular BeanPostProcessors.
   		List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
   		for (String ppName : nonOrderedPostProcessorNames) {
   			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
   			nonOrderedPostProcessors.add(pp);
   			if (pp instanceof MergedBeanDefinitionPostProcessor) {
   				internalPostProcessors.add(pp);
   			}
   		}
   		registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);
   
   		// Finally, re-register all internal BeanPostProcessors.
   		sortPostProcessors(internalPostProcessors, beanFactory);
   		registerBeanPostProcessors(beanFactory, internalPostProcessors);
   
   		// Re-register post-processor for detecting inner beans as ApplicationListeners,
   		// moving it to the end of the processor chain (for picking up proxies etc).
   		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
   	}
   ```

   

11. 

12. 

    

Spring有一个Compartor初始化的时候如果有执行顺序的问题可能会用到这个

设置autowireCandidateResolver，候选者解析器,这里面可以判断lazy，如果是lazy构造一个代理对象，他爹是用来判断qulifier

**ConfigurationClassPostProcessor** 生成beandefinition

判断用没用JPA，EVENTPOSTPROCESSOR，等等都是spring内置的bean

### 什么是beanFactoryPostProcessor

拿到beanfactory （这里面有比较器，解析器，还有一些beandefinition，就是前面初始化的一共6个，有3个是beanfactory的后置处理器，2个是bean的后置处理器（autowired，common），就这些东西，单例池什么的还没初始化）

1.执行beanfacotry的后置处理器，主要是扫描

2.初始化事件发布器，初始化监听器

3.创建非懒加载的单例bean

### refresh方法

1.prepare() 记一个开始事件，状态激活等

2.initPropertySource() 这个方法给子类用，如果希望把配置放到环境变量里面，那么可以实现这个方法，比如把**servletContext**放里面

3.environment().validate() 验证环境变量里的key，value





