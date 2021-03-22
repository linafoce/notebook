## spring

### XML的autowire自动注入

在XML中，我们可以在定义一个Bean时去指定这个Bean的自动注入模式：

1. byType
2. byName
3. constructor
4. default
5. no

比如：

```xml
<bean id="userService" class="com.luban.service.UserService" autowire="byType"/>
```

这么写，表示Spring会自动的给userService中所有的属性自动赋值（**不需要**这个属性上有@Autowired注解，但需要这个属性有对应的**set方法**）。

在创建Bean的过程中，在填充属性时，Spring会去解析当前类，把当前类的所有方法都解析出来，Spring会去解析每个方法得到对应的PropertyDescriptor对象，注意PropertyDescriptor并不是Spring中的类，而是java.beans包下类，也就是jdk自带的类，PropertyDescriptor中有几个属性：

1. **name：这个name并不是方法的名字，而是拿方法名字进过处理后的名字**

1. 1. **如果方法名字以“get”开头，比如“getXXX”,那么name=XXX**
   2. **如果方法名字以“is”开头，比如“isXXX”,那么name=XXX**
   3. **如果方法名字以“set”开头，比如“setXXX”,那么name=XXX**

1. **readMethodRef：表示get方法的Method对象的引用**
2. **readMethodName：表示get方法的名字**
3. **writeMethodRef：****表示set方法的Method对象的引用**
4. **writeMethodName：****表示set方法的名字**
5. **propertyTypeRef：如果有get方法那么对应的就是返回值的类型，如果是set方法那么对应的就是set方法中唯一参数的类型**

get方法的定义是： 方法参数个数为0个，并且 （方法名字以"get"开头 或者 方法名字以"is"开头并且方法的返回类型为boolean）

set方法的定义是：方法参数个数为1个，并且 （方法名字以"set"开头并且方法返回类型为void）

所以，Spring在通过byName的自动填充属性时流程是：

1. 找到所有set方法所对应的XXX部分的名字
2. 根据XXX部分的名字去获取bean



##### Spring在通过byType的自动填充属性时流程是：

1. 找到所有set方法所对应的XXX部分的名字
2. 根据XXX部分的名字重新再获取得到PropertyDescriptor
3. 获取到set方法中的唯一参数的类型，并且根据该类型去容器中获取bean
4. 如果找到多个，会报错。



以上，分析了autowire的byType和byName情况，那么接下来分析constructor，constructor表示通过构造方法注入，其实这种情况就比较简单了，没有byType和byName那么复杂。



首先，如果是byType或byName，那么该bean一定要有一个无参的构造方法，因为如果只有有参构造方法，那么Spring将无法进行实例化，因为Spring如果要实例化肯定需要利用构造方法，而Spring没法给构造方法传值。



然后，如果是constructor，那么就可以不写set方法了，当某个bean是通过构造方法来注入时，表示Spring在利用构造方法实例化一个对象时，可以利用构造方法的参数信息从Spring容器中去找bean，找到bean之后作为参数传给构造方法，从而实例化得到一个bean对象。

我们这里先不考虑一个类有多个构造方法的情况，后面单独讲**推断构造方法**。我们这里只考虑只有一个有参构造方法。

其实构造方法注入相当于byType+byName，普通的byType是根据set方法中的参数类型去找bean，找到多个会报错，而constructor就是通过构造方法中的参数类型去找bean，如果找到多个会根据参数名确定。



另外两个：

1. no，表示关闭autowire
2. default，表示默认值，我们一直演示的某个bean的autowire，而也可以直接在<beans>标签中设置autowire，如果设置了，那么<bean>标签中设置的autowire如果为default，那么则会用<beans>标签中设置的autowire。

可以发现XML中的自动注入是挺强大的，那么问题来了，**为什么我们平时都是用的@Autowired注解呢？而没有用上文说的这种自动注入方式呢？**

@Autowired注解相当于XML中的autowire属性的**注解方式的替代**。这是在官网上有提到的。



## Spring几种依赖注入方式

### 1. 手动注入

1. set

   ```xml
   <bean name="userService" class="com.luban.UserService">
     <property name="orderService" ref="orderService">
   </bean>  
   ```

2. 构造方法

   ```xml
   <bean name="userService" class="com.luban.UserService">
     <constructor index="0" ref="orderService">
   </bean> 
   ```

### 2. 自动注入

1. xml里面的自动注入 

   本质set方法，构造方法

```xml
 <!-- bytype是去找set方法，获得里面的类型OrderService,所以set方法运行的时候一定会执行 -->
<bean name="userService" class="com.luban.UserService" autowire="bytype"/>

 <!-- bytype是去找set方法，获得setOrderService(OrderService orderService),set之后的这个名字 -->
<bean name="userService" class="com.luban.UserService" autowire="byname"/>

<!--通过推断构造方法 -->
<bean name="userService" class="com.luban.UserService" autowire="constructor"/>

<!--不自动注入 -->
<bean name="userService" class="com.luban.UserService" autowire="no"/>

<!--在beans标签里面来，就是xml开头,说白了就是头部一旦配置，相当于所有的bean都是用这一种注入方式 -->
<bean name="userService" class="com.luban.UserService" autowire="default"/>

<!-- 要使用autowired还要打开开关，XMLApplicationContext是没有AutoWired的后置处理器-->
<context:annotation-config></context:annotation-config>
```

2. @Autowired的自动注入(用途更广)

   applyMergedBeanDefinnitionPostProcessor（）这个方法会查找autowire的注入点

   1. set方法

   2. 属性

      先bytype后byname

   3. 构造方法

      

BeanDefinition

1. propertyValues( Map)  key属性的名字，value 属性的值

## 这里面如果走的xml配置，里面设置了byname或者bytype

## 否则就解析autowired属性

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		if (bw == null) {
			if (mbd.hasPropertyValues()) {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
			}
			else {
				// Skip property population phase for null instance.
				return;
			}
		}

		// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
		// 可以提供InstantiationAwareBeanPostProcessor，控制对象的属性注入
		// 我们可以自己写一个InstantiationAwareBeanPostProcessor，然后重写postProcessAfterInstantiation方法返回false,那么则不会进行属性填充了
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
						return;
					}
				}
			}
		}

		// 是否在BeanDefinition中设置了属性值
		// @autowired注解中默认是no的，这里面一般是不走的
		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			// by_name是根据根据属性名字找bean
			// by_type是根据属性所对应的set方法的参数类型找bean
			// 找到bean之后都要调用set方法进行注入

			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// Add property values based on autowire by name if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;

			// 总结一下
			// 其实就是Spring自动的根据某个类中的set方法来找bean，byName就是根据某个set方法所对应的属性名去找bean
			// byType，就是根据某个set方法的参数类型去找bean
			// 注意，执行完这里的代码之后，这是把属性以及找到的值存在了pvs里面，并没有完成反射赋值
		}

  	// 大多都会进入这一段逻辑
		// 执行完了Spring的自动注入之后，就开始解析@Autowired，这里叫做实例化回调
		// autowired功能是通过bean的后置处理器实现的
		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;

					// 调用BeanPostProcessor分别解析@Autowired、@Resource、@Value，得到属性值
          // 因为autowired其实是调用一个后置处理器
					PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);

					if (pvsToUse == null) {
						if (filteredPds == null) {
							filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
						}
						pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
						if (pvsToUse == null) {
							return;
						}
					}
					pvs = pvsToUse;
				}
			}
		}
		if (needsDepCheck) {
			if (filteredPds == null) {
				filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
			}
			checkDependencies(beanName, mbd, filteredPds, pvs);
		}

		if (pvs != null) {
			// pvs存的就是属性已经对应的值
      // 如果不是autowired，或者自动注入，是通过xml手动指定的，那么就会直接走这个逻辑
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
	}
```

## Autowired逻辑

## 核心类 AutowiredAnnotationBeanPostProcessor

### 两个核心方法,这个在实例化后调用 postProcessMergedBeanDefinition

```java
// 这个是继承meragePostProcessor中的方法
// 在docreateBean方法中，实例化后会走这个逻辑，查找注入点
// 注入点就是加了@autowired的地方
@Override
	public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
		// 获取beanType中的注入点
    // 这个是一个注入点的集合
		InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
		metadata.checkConfigMembers(beanDefinition);
	}


// 寻找注入点的方法
private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, @Nullable PropertyValues pvs) {
		// Fall back to class name as cache key, for backwards compatibility with custom callers.
		String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());
		// Quick check on the concurrent map first, with minimal locking.
		InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
		if (InjectionMetadata.needsRefresh(metadata, clazz)) {
			synchronized (this.injectionMetadataCache) {
				metadata = this.injectionMetadataCache.get(cacheKey);
				if (InjectionMetadata.needsRefresh(metadata, clazz)) {
					if (metadata != null) {
						metadata.clear(pvs);
					}
					// 寻找当前clazz中的注入点，把所有注入点整合成为一个InjectionMetadata对象
					metadata = buildAutowiringMetadata(clazz);
					this.injectionMetadataCache.put(cacheKey, metadata);
				}
			}
		}
		return metadata;
	}

// 整合原数据的方法
private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {

		// 判断是不是候选者类，比如说类名，如果是以"java."开头的则不是候选者类
		if (!AnnotationUtils.isCandidateClass(clazz, this.autowiredAnnotationTypes)) {
			return InjectionMetadata.EMPTY;
		}

		//
		List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
		Class<?> targetClass = clazz;

		do {
			final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();

			// 遍历属性，看是否有@Autowired，@Value，@Inject注解
			ReflectionUtils.doWithLocalFields(targetClass, field -> {
				MergedAnnotation<?> ann = findAutowiredAnnotation(field);
				// 如果存在@Autowired，@Value，@Inject注解其中一个
				if (ann != null) {
					// 如果字段是static的，则直接进行返回，不进行注入
					if (Modifier.isStatic(field.getModifiers())) {
						if (logger.isInfoEnabled()) {
							logger.info("Autowired annotation is not supported on static fields: " + field);
						}
						return;
					}
					// 是否required
					boolean required = determineRequiredStatus(ann);
					// 生成一个注入点AutowiredFieldElement
					currElements.add(new AutowiredFieldElement(field, required));
				}
			});

			// 遍历方法，看是否有@Autowired，@Value，@Inject注解
			ReflectionUtils.doWithLocalMethods(targetClass, method -> {
				Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
				if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
					return;
				}
				MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);
				if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
					// 静态方法不能用来注入属性
					if (Modifier.isStatic(method.getModifiers())) {
						if (logger.isInfoEnabled()) {
							logger.info("Autowired annotation is not supported on static methods: " + method);
						}
						return;
					}
					// 方法参数值为0，不能用来注入属性
					if (method.getParameterCount() == 0) {
						if (logger.isInfoEnabled()) {
							logger.info("Autowired annotation should only be used on methods with parameters: " +
									method);
						}
					}
					// 看是否有@required注解
          // 是不是必须注入的
					boolean required = determineRequiredStatus(ann);
					// 根据方法找出对应的属性
					PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
					currElements.add(new AutowiredMethodElement(method, required, pd));
				}
			});

			// 所有能够注入的属性集合
			elements.addAll(0, currElements);
			targetClass = targetClass.getSuperclass();
		}
		while (targetClass != null && targetClass != Object.class);

		// 所有能够注入的属性集合对应的InjectionMetadata对象
		return InjectionMetadata.forElements(elements, clazz);
	}


// 查询注解的方法
@Nullable
	private MergedAnnotation<?> findAutowiredAnnotation(AccessibleObject ao) {
		// 查看当前字段上是否存在@Autowired，@Value，@Inject注解，存在其中一个则返回，表示可以注入
		MergedAnnotations annotations = MergedAnnotations.from(ao);
		// autowiredAnnotationTypes是一个LinkedHashSet，所以会按顺序去判断当前字段中是否有Autowired注解，如果有则返回
		// 如果没有Autowired注解，那么则判断是否有Value注解，在判断是否有Inject注解
		for (Class<? extends Annotation> type : this.autowiredAnnotationTypes) {
			MergedAnnotation<?> annotation = annotations.get(type);
			if (annotation.isPresent()) {
				return annotation;
			}
		}
		return null;
	}
```

### 填充属性时执行的方法

```java
@Override
	public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
		// InjectionMetadata中保存了所有被@Autowired注解标注的属性/方法并封装成一个个的InjectedElement
		InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
		try {
			// 按注入点进行注入
			metadata.inject(bean, beanName, pvs);
		}
		catch (BeanCreationException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
		}
		return pvs;
	}

// 注入的方法
// metadata里面有两个字类，他们都重写了inject方法
@Override
		protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
			Field field = (Field) this.member;
			Object value;
			if (this.cached) {
				// 当前注入点已经注入过了，有缓存了，则利用cachedFieldValue去找对应的bean
				value = resolvedCachedArgument(beanName, this.cachedFieldValue);
			}
			else {
				//  Spring在真正查找属性对应的对象之前, 会先将该属性的描述封装成一个DependencyDescriptor, 里面保存了Filed、是否强制需要即required, 以及属性所在的类(即Field所在的类Class对象)
				DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
				desc.setContainingClass(bean.getClass());
				Set<String> autowiredBeanNames = new LinkedHashSet<>(1);
				Assert.state(beanFactory != null, "No BeanFactory available");
				TypeConverter typeConverter = beanFactory.getTypeConverter();
				try {
					// 根据field去寻找合适的bean
					value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
				}
				catch (BeansException ex) {
					throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(field), ex);
				}
				synchronized (this) {
					if (!this.cached) {
						if (value != null || this.required) {
							this.cachedFieldValue = desc;
							// 注册当前bean依赖了哪些其他的bean的name
							registerDependentBeans(beanName, autowiredBeanNames);
							if (autowiredBeanNames.size() == 1) {
								String autowiredBeanName = autowiredBeanNames.iterator().next();
								if (beanFactory.containsBean(autowiredBeanName) &&
										beanFactory.isTypeMatch(autowiredBeanName, field.getType())) {
									// 对得到的对象进行缓存
									this.cachedFieldValue = new ShortcutDependencyDescriptor(
											desc, autowiredBeanName, field.getType());
								}
							}
						}
						else {
							this.cachedFieldValue = null;
						}
						this.cached = true;
					}
				}
			}
			// 反射设值
			if (value != null) {
				ReflectionUtils.makeAccessible(field);
				field.set(bean, value);
			}
		}
	}


// 如果注解加在方法上会走这个方法
@Override
		protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
			if (checkPropertySkipping(pvs)) {
				return;
			}
			Method method = (Method) this.member;
			Object[] arguments;
			if (this.cached) {
				// Shortcut for avoiding synchronization...
				arguments = resolveCachedArguments(beanName);
			}
			else {
				int argumentCount = method.getParameterCount();
				arguments = new Object[argumentCount];
				DependencyDescriptor[] descriptors = new DependencyDescriptor[argumentCount];

				// 记录自动注入的beanName
				Set<String> autowiredBeans = new LinkedHashSet<>(argumentCount);
				Assert.state(beanFactory != null, "No BeanFactory available");
				TypeConverter typeConverter = beanFactory.getTypeConverter();
				// 遍历当前set方法中的每个参数，将方法参数
				for (int i = 0; i < arguments.length; i++) {
					// 方法参数对象
					MethodParameter methodParam = new MethodParameter(method, i);
					DependencyDescriptor currDesc = new DependencyDescriptor(methodParam, this.required);
					currDesc.setContainingClass(bean.getClass());
					descriptors[i] = currDesc;
					try {
						// 寻找bean
						Object arg = beanFactory.resolveDependency(currDesc, beanName, autowiredBeans, typeConverter);
						if (arg == null && !this.required) {
							arguments = null;
							break;
						}
						arguments[i] = arg;
					}
					catch (BeansException ex) {
						throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(methodParam), ex);
					}
				}

				// arguments中存储的就是所找到的bean对象，构造为ShortcutDependencyDescriptor进行缓存
				synchronized (this) {
					if (!this.cached) {
						if (arguments != null) {
							DependencyDescriptor[] cachedMethodArguments = Arrays.copyOf(descriptors, arguments.length);
							registerDependentBeans(beanName, autowiredBeans);

							if (autowiredBeans.size() == argumentCount) {
								Iterator<String> it = autowiredBeans.iterator();
								Class<?>[] paramTypes = method.getParameterTypes();
								for (int i = 0; i < paramTypes.length; i++) {
									String autowiredBeanName = it.next();
									if (beanFactory.containsBean(autowiredBeanName) &&
											beanFactory.isTypeMatch(autowiredBeanName, paramTypes[i])) {
										cachedMethodArguments[i] = new ShortcutDependencyDescriptor(
												descriptors[i], autowiredBeanName, paramTypes[i]);
									}
								}
							}
							this.cachedMethodArguments = cachedMethodArguments;
						}
						else {
							this.cachedMethodArguments = null;
						}
						this.cached = true;
					}
				}
			}
			if (arguments != null) {
				try {
					ReflectionUtils.makeAccessible(method);
					method.invoke(bean, arguments);
				}
				catch (InvocationTargetException ex) {
					throw ex.getTargetException();
				}
			}
		}
```

## spring如何在后置处理其中找到一个bean

## 核心方法 resolveDependency

```java

/**
*
*
*
这个方法只负责找对象，找完之后对象返回出去
*/
	@Override
	@Nullable
	public Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName,
			@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {
		// DependencyDescriptor表示一个依赖，可以是一个属性字段，可能是一个构造方法参数，可能是一个set方法参数
		// 根据descriptor去BeanFactory中找到bean

		descriptor.initParameterNameDiscovery(getParameterNameDiscoverer());

		// 如果依赖的类型是Optional
		if (Optional.class == descriptor.getDependencyType()) {
			return createOptionalDependency(descriptor, requestingBeanName);
		}
		else if (ObjectFactory.class == descriptor.getDependencyType() ||
				ObjectProvider.class == descriptor.getDependencyType()) {
			return new DependencyObjectProvider(descriptor, requestingBeanName);
		}
		else if (javaxInjectProviderClass == descriptor.getDependencyType()) {
			return new Jsr330Factory().createDependencyProvider(descriptor, requestingBeanName);
		}
		else {

			Object result = getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(
					descriptor, requestingBeanName);

			if (result == null) {
				// 通过解析descriptor找到bean对象
				result = doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);
			}
			return result;
		}
	}
```

## 再次往下面走

```java
@Nullable
	public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
			@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

		InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
		try {
			// 如果DependencyDescriptor是一个ShortcutDependencyDescriptor，
			// 那么会直接理解beanName从beanFactory中拿到一个bean，
			// 在利用@Autowired注解来进行依赖注入时会利用ShortcutDependencyDescriptor来进行依赖注入的缓存，
			// 表示当解析完某个依赖信息后，会把依赖的bean的beanName缓存起来
			Object shortcut = descriptor.resolveShortcut(this);
			if (shortcut != null) {
				return shortcut;
			}

			// 获取descriptor具体的类型，某个Filed的类型或某个方法参数的类型
			Class<?> type = descriptor.getDependencyType();

			// 获取@Value注解中所配置的值
			Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor); // 是否通过@Value注解指定了值
			if (value != null) {
				if (value instanceof String) {
					// 先进行占位符的填充，解析"$"符号
					String strVal = resolveEmbeddedValue((String) value);

					BeanDefinition bd = (beanName != null && containsBean(beanName) ?
							getMergedBeanDefinition(beanName) : null);

					// 解析Spring EL表达式，解析"#"符号（可以进行运算，可以写某个bean的名字）
					value = evaluateBeanDefinitionString(strVal, bd);
				}
				TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
				try {
					// 把value从descriptor.getTypeDescriptor()类型转化为type类
					return converter.convertIfNecessary(value, type, descriptor.getTypeDescriptor());
				}
				catch (UnsupportedOperationException ex) {
					// A custom TypeConverter which does not support TypeDescriptor resolution...
					return (descriptor.getField() != null ?
							converter.convertIfNecessary(value, type, descriptor.getField()) :
							converter.convertIfNecessary(value, type, descriptor.getMethodParameter()));
				}
			}

			// 没有使用@Value注解

			// 要注入的依赖的类型是不是一个Map、Array、Collection
			Object multipleBeans = resolveMultipleBeans(descriptor, beanName, autowiredBeanNames, typeConverter);
			if (multipleBeans != null) {
				return multipleBeans;
			}

			// 通过type找，可能找到多个
			Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
			if (matchingBeans.isEmpty()) {
				if (isRequired(descriptor)) {
					raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
				}
				return null;
			}

			String autowiredBeanName;
			Object instanceCandidate;

			// 根据type找到了多个
			if (matchingBeans.size() > 1) {
				autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
				if (autowiredBeanName == null) {
					if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
						return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
					}
					else {
						// In case of an optional Collection/Map, silently ignore a non-unique case:
						// possibly it was meant to be an empty collection of multiple regular beans
						// (before 4.3 in particular when we didn't even look for collection beans).
						return null;
					}
				}
				instanceCandidate = matchingBeans.get(autowiredBeanName);
			}
			else {
				// We have exactly one match.
				Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
				autowiredBeanName = entry.getKey();
				instanceCandidate = entry.getValue();
			}

			if (autowiredBeanNames != null) {
				autowiredBeanNames.add(autowiredBeanName);
			}
			if (instanceCandidate instanceof Class) {
				// 调用beanFactory.getBean(beanName);创建bean对象
				instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
			}
			Object result = instanceCandidate;
			if (result instanceof NullBean) {
				if (isRequired(descriptor)) {
					raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
				}
				result = null;
			}
			if (!ClassUtils.isAssignableValue(type, result)) {
				throw new BeanNotOfRequiredTypeException(autowiredBeanName, type, instanceCandidate.getClass());
			}
			return result;
		}
		finally {
			ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
		}
	}

```











**byName**

1. 找到所有set方法所对应的XXX部分的名字
2. 根据XXX部分的名字去获取bean
3. 看看是否有beanName,如果没有就走getBean逻辑,doCreateBean创建
4. 加入propertyValues里面

```java
/**
	 * Fill in any missing property values with references to
	 * other beans in this factory if autowire is set to "byName".
	 * @param beanName the name of the bean we're wiring up.
	 * Useful for debugging messages; not used functionally.
	 * @param mbd bean definition to update through autowiring
	 * @param bw the BeanWrapper from which we can obtain information about the bean
	 * @param pvs the PropertyValues to register wired objects with
	  如果是byname的方式, 比如xml指定 autowire="byname"
	 */
	protected void autowireByName(
			String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {

		// 找到有对应set方法的属性
		String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
		for (String propertyName : propertyNames) {
			if (containsBean(propertyName)) {
				// 根据属性名去找bean，这就是byName
				Object bean = getBean(propertyName);
				// 给属性赋值
				pvs.add(propertyName, bean);
				registerDependentBean(propertyName, beanName);
				if (logger.isTraceEnabled()) {
					logger.trace("Added autowiring by name from bean name '" + beanName +
							"' via property '" + propertyName + "' to bean named '" + propertyName + "'");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Not autowiring property '" + propertyName + "' of bean '" + beanName +
							"' by name: no matching bean found");
				}
			}
		}
	}
```







**byType**

1. 先拿名字，跟byName一样 unsatisfiedNonSimpleProperties()方法
2. 拿PropertyDescriptor[]， 描述，比如参数的类型，对象的引用
3. 拿所有的方法，有的是反射,判断是不是set，get方法，is会忽略,判断是不是有形参,拿了名字大写转小些

1. 注入点 --- > 只有set方法
2. Autowired注入点 xxx(xxx,xxx) 任何方法，只要你加Autowired
   1. 基于注入点注入， 如果是Field --》 封装成 DependencyDescriptor
   2. 如果是Metthod --〉 每个参数都封装成DependencyDescriptor
   3. resolveDependency解析依赖 ---> 就走getBean逻辑

```java
/**
	 * Abstract method defining "autowire by type" (bean properties by type) behavior.
	 * <p>This is like PicoContainer default, in which there must be exactly one bean
	 * of the property type in the bean factory. This makes bean factories simple to
	 * configure for small namespaces, but doesn't work as well as standard Spring
	 * behavior for bigger applications.
	 * @param beanName the name of the bean to autowire by type
	 * @param mbd the merged bean definition to update through autowiring
	 * @param bw the BeanWrapper from which we can obtain information about the bean
	 * @param pvs the PropertyValues to register wired objects with
	 */
	protected void autowireByType(
			String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {

		TypeConverter converter = getCustomTypeConverter();
		if (converter == null) {
			converter = bw;
		}

		Set<String> autowiredBeanNames = new LinkedHashSet<>(4);

		// 找到有对应set方法的属性
    // 先把名字拿出来
		String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
		for (String propertyName : propertyNames) {
			try {
				PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);
				// Don't try autowiring by type for type Object: never makes sense,
				// even if it technically is a unsatisfied, non-simple property.
        // 因为是bytype如果是object类型，找到的是所有的类
				if (Object.class != pd.getPropertyType()) {
					// set方法中的参数信息
					MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd);
					// Do not allow eager init for type matching in case of a prioritized post-processor.
					boolean eager = !(bw.getWrappedInstance() instanceof PriorityOrdered);
					DependencyDescriptor desc = new AutowireByTypeDependencyDescriptor(methodParam, eager);

					// 根据类型找bean，这就是byType
					Object autowiredArgument = resolveDependency(desc, beanName, autowiredBeanNames, converter);
					if (autowiredArgument != null) {
						pvs.add(propertyName, autowiredArgument);
					}
					for (String autowiredBeanName : autowiredBeanNames) {
						registerDependentBean(autowiredBeanName, beanName);
						if (logger.isTraceEnabled()) {
							logger.trace("Autowiring by type from bean name '" + beanName + "' via property '" +
									propertyName + "' to bean named '" + autowiredBeanName + "'");
						}
					}
					autowiredBeanNames.clear();
				}
			}
			catch (BeansException ex) {
				throw new UnsatisfiedDependencyException(mbd.getResourceDescription(), beanName, propertyName, ex);
			}
		}
	}
```

## unsatisfiedNonSimpleProperties

```java
/**
	 * Return an array of non-simple bean properties that are unsatisfied.
	 * These are probably unsatisfied references to other beans in the
	 * factory. Does not include simple properties like primitives or Strings.
	 * @param mbd the merged bean definition the bean was created with
	 * @param bw the BeanWrapper the bean was created with
	 * @return an array of bean property names
	 * @see org.springframework.beans.BeanUtils#isSimpleProperty
	 */
	protected String[] unsatisfiedNonSimpleProperties(AbstractBeanDefinition mbd, BeanWrapper bw) {
		Set<String> result = new TreeSet<>();
		PropertyValues pvs = mbd.getPropertyValues(); // 在BeanDefinition中添加的属性和值，
    // 这个类里面的属性记录了，set方法，get方法，方法的参数
		PropertyDescriptor[] pds = bw.getPropertyDescriptors();

		// 对类里所有的属性进行过滤,确定哪些属性是需要进行自动装配的
		for (PropertyDescriptor pd : pds) {
			// 属性有set方法，并且
			// 没有通过DependencyCheck排除，并且
			// 没有在BeanDefinition中给该属性赋值，并且
			// 属性的类型不是简单类型
			if (pd.getWriteMethod() != null && !isExcludedFromDependencyCheck(pd) && !pvs.contains(pd.getName()) &&
					!BeanUtils.isSimpleProperty(pd.getPropertyType())) {
				result.add(pd.getName());
			}
		}

		// 返回过滤之后的结果，后续只对这些属性进行自动装配
		return StringUtils.toStringArray(result);
	}
```

## getPropertyDescriptors

```java
@Override
// 如何获取属性的修饰呢
	public PropertyDescriptor[] getPropertyDescriptors() {
		return getCachedIntrospectionResults().getPropertyDescriptors();
	}

// 从缓存里拿
private CachedIntrospectionResults getCachedIntrospectionResults() {
		if (this.cachedIntrospectionResults == null) {
			this.cachedIntrospectionResults = CachedIntrospectionResults.forClass(getWrappedClass());
		}
		return this.cachedIntrospectionResults;
	}

// 这里是从构造方法里面拿
static CachedIntrospectionResults forClass(Class<?> beanClass) throws BeansException {
		CachedIntrospectionResults results = strongClassCache.get(beanClass);
		if (results != null) {
			return results;
		}
		results = softClassCache.get(beanClass);
		if (results != null) {
			return results;
		}

		results = new CachedIntrospectionResults(beanClass);
		ConcurrentMap<Class<?>, CachedIntrospectionResults> classCacheToUse;

		if (ClassUtils.isCacheSafe(beanClass, CachedIntrospectionResults.class.getClassLoader()) ||
				isClassLoaderAccepted(beanClass.getClassLoader())) {
			classCacheToUse = strongClassCache;
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("Not strongly caching class [" + beanClass.getName() + "] because it is not cache-safe");
			}
			classCacheToUse = softClassCache;
		}

		CachedIntrospectionResults existing = classCacheToUse.putIfAbsent(beanClass, results);
		return (existing != null ? existing : results);
	}


// 关键是getBeanInfo
private CachedIntrospectionResults(Class<?> beanClass) throws BeansException {
		try {
			if (logger.isTraceEnabled()) {
				logger.trace("Getting BeanInfo for class [" + beanClass.getName() + "]");
			}
      // 这里面会把setUserService 名字做处理
      // 去掉set 变成 userService
			this.beanInfo = getBeanInfo(beanClass);

			if (logger.isTraceEnabled()) {
				logger.trace("Caching PropertyDescriptors for class [" + beanClass.getName() + "]");
			}
			this.propertyDescriptorCache = new LinkedHashMap<>();

			// This call is slow so we do it once.
			PropertyDescriptor[] pds = this.beanInfo.getPropertyDescriptors();
			for (PropertyDescriptor pd : pds) {
				if (Class.class == beanClass &&
						("classLoader".equals(pd.getName()) ||  "protectionDomain".equals(pd.getName()))) {
					// Ignore Class.getClassLoader() and getProtectionDomain() methods - nobody needs to bind to those
					continue;
				}
				if (logger.isTraceEnabled()) {
					logger.trace("Found bean property '" + pd.getName() + "'" +
							(pd.getPropertyType() != null ? " of type [" + pd.getPropertyType().getName() + "]" : "") +
							(pd.getPropertyEditorClass() != null ?
									"; editor [" + pd.getPropertyEditorClass().getName() + "]" : ""));
				}
				pd = buildGenericTypeAwarePropertyDescriptor(beanClass, pd);
				this.propertyDescriptorCache.put(pd.getName(), pd);
			}

			// Explicitly check implemented interfaces for setter/getter methods as well,
			// in particular for Java 8 default methods...
			Class<?> currClass = beanClass;
			while (currClass != null && currClass != Object.class) {
				introspectInterfaces(beanClass, currClass);
				currClass = currClass.getSuperclass();
			}

			this.typeDescriptorCache = new ConcurrentReferenceHashMap<>();
		}
		catch (IntrospectionException ex) {
			throw new FatalBeanException("Failed to obtain BeanInfo for class [" + beanClass.getName() + "]", ex);
		}
	}
```



