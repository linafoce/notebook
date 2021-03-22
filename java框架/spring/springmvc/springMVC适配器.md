# 【11.22】HandlerMapping

默认情况下，Spring MVC会加载在当前系统中所有实现了HandlerMapping接口的bean，再进行按优先级排序。如果只期望Spring MVC只加载指定的HandlerMapping，可以修改web.xml中的DispatcherServlet的初始化参数，将detectAllHandlerMappings的值设置为false。这样，Spring MVC就只会查找名为“handlerMapping”的bean，并作为当前系统的唯一的HandlerMapping。所以在DispatcherServlet的initHandlerMappings()方法中，优先判断detectAllHandlerMappings的值，如果没有定义HandlerMapping的话，Spring MVC就会按照DispatcherServlet.properties所定义的内容来加载默认的HandlerMapping

- SimpleUrlHandlerMapping 支持映射bean实例和映射bean名称，需要手工维护urlmap，通过key指定访问路径，
- BeanNameUrlHandlerMapping 支持映射bean的name属性值，扫描以”/“开头的beanname，通过id指定访问路径
- RequestMappingHandlerMapping  支持@Controller和@RequestMapping注解，通过注解定义访问路径

作用是将请求映射到处理程序，以及预处理和处理后的拦截器列表，映射是基于一些标准的，其中的细节因不同的实现而不相同。这是官方文档上一段描述，该接口只有一个方法getHandler(request)，返回一个HandlerExecutionChain对象

```java
public interface HandlerMapping {
    // 返回请求的一个处理程序handler和拦截器interceptors
    @Nullable
    HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```

### initHandlerMappings

```java
private void initHandlerMappings(ApplicationContext context) {
    this.handlerMappings = null;
    //默认为true，可通过DispatcherServlet的init-param参数进行设置
    if (this.detectAllHandlerMappings) {
        //在ApplicationContext中找到所有的handlerMapping, 包括父级上下文
        Map<String, HandlerMapping> matchingBeans =
          	//这个方法会从ioc里面加载HandlerMapping的
            BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList<>(matchingBeans.values());
           //排序，可通过指定order属性进行设置，order的值为int型，数越小优先级越高
            AnnotationAwareOrderComparator.sort(this.handlerMappings);
        }
    } else {
        try {
            //从ApplicationContext中取id（或name）="handlerMapping"的bean,此时为空
            HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
            // 将hm转换成list，并赋值给属性handlerMappings
            this.handlerMappings = Collections.singletonList(hm);
        } catch (NoSuchBeanDefinitionException ex) {
        }
    }

    //如果没有自定义则使用默认的handlerMappings
    //默认的HandlerMapping在DispatcherServlet.properties属性文件中定义，
    // 该文件是在DispatcherServlet的static静态代码块中加载的
    // 默认的是：BeanNameUrlHandlerMapping和RequestMappingHandlerMapping
    if (this.handlerMappings == null) {
        this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);//默认策略
        if (logger.isTraceEnabled()) {
            logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
                         "': using default strategies from DispatcherServlet.properties");
        }
    }
}


protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {
   String key = strategyInterface.getName();//HandlerMappings
    //获取HandlerMappings为key的value值,defaultStrategies在static块中读取DispatcherServlet.properties
   String value = defaultStrategies.getProperty(key);
   if (value != null) {
       //逗号，分割成数组
      String[] classNames = StringUtils.commaDelimitedListToStringArray(value);
      List<T> strategies = new ArrayList<>(classNames.length);
      for (String className : classNames) {
         try {
             //反射加载并存储strategies
            Class<?> clazz = ClassUtils.forName(className, DispatcherServlet.class.getClassLoader());
             //通过容器创建bean 触发后置处理器    
            Object strategy = createDefaultStrategy(context, clazz);
            strategies.add((T) strategy);
         }
         。。。
      }
      return strategies;
   }
   else {
      return new LinkedList<>();
   }
}

protected Object createDefaultStrategy(ApplicationContext context, Class<?> clazz) {
    //通过容器创建bean
    return context.getAutowireCapableBeanFactory().createBean(clazz);
}
```

DispatcherServlet.properties中的三个HandlerMapping

##### AbstractHandlerMapping

模板类

```
protected void initApplicationContext() throws BeansException {
    // 提供给子类去重写的，不过Spring并未去实现,提供扩展
    extendInterceptors(this.interceptors);
    // 加载拦截器
    detectMappedInterceptors(this.adaptedInterceptors);
    // 归并拦截器
    initInterceptors();
}

/**
 * 空实现
 */
protected void extendInterceptors(List<Object> interceptors) {
}

/**
 * 从上下文中加载MappedInterceptor类型的拦截器，比如我们在配置文件中使用
 * <mvc:interceptors></mvc:interceptors>
 * 标签配置的拦截器
 */
protected void detectMappedInterceptors(List<HandlerInterceptor> mappedInterceptors) {
    mappedInterceptors.addAll(
            BeanFactoryUtils.beansOfTypeIncludingAncestors(
                    obtainApplicationContext(), MappedInterceptor.class, true, false).values());
}

/**
 * 合并拦截器，即将<mvc:interceptors></mvc:interceptors>中的拦截器与HandlerMapping中通过属性interceptors设置的拦截器进行合并
 */
protected void initInterceptors() {
    if (!this.interceptors.isEmpty()) {
        for (int i = 0; i < this.interceptors.size(); i++) {
            Object interceptor = this.interceptors.get(i);
            if (interceptor == null) {
                throw new IllegalArgumentException("Entry number " + i + " in interceptors array is null");
            }
            // 适配后加入adaptedInterceptors
            this.adaptedInterceptors.add(adaptInterceptor(interceptor));
        }
    }
}

/**
 * 适配HandlerInterceptor和WebRequestInterceptor
 */
protected HandlerInterceptor adaptInterceptor(Object interceptor) {
    if (interceptor instanceof HandlerInterceptor) {
        return (HandlerInterceptor) interceptor;
    } else if (interceptor instanceof WebRequestInterceptor) {
        return new WebRequestHandlerInterceptorAdapter((WebRequestInterceptor) interceptor);
    } else {
        throw new IllegalArgumentException("Interceptor type not supported: " +                                                                                 interceptor.getClass().getName());
    }
}

/**
 * 返回请求处理的HandlerExecutionChain，从AbstractHandlerMapping中的adaptedInterceptors和mappedInterceptors属性中获取
 */
@Override
@Nullable
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    // getHandlerInternal()为抽象方法，具体需子类实现
    Object handler = getHandlerInternal(request);
    if (handler == null) {
        handler = getDefaultHandler();
    }
    if (handler == null) {
        return null;
    }
    // Bean name or resolved handler?
    if (handler instanceof String) {
        String handlerName = (String) handler;
        handler = obtainApplicationContext().getBean(handlerName);
    }
    
    // 将请求处理器封装为HandlerExectionChain
    HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
    // 对跨域的处理
    if (CorsUtils.isCorsRequest(request)) {
        CorsConfiguration globalConfig = this.globalCorsConfigSource.getCorsConfiguration(request);
        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
        CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);
        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
    }
    return executionChain;
}

/**
 * 钩子函数，需子类实现
 */
@Nullable
protected abstract Object getHandlerInternal(HttpServletRequest request) throws Exception;

/**
 * 构建handler处理器的HandlerExecutionChain，包括拦截器
 */
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
    HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
            (HandlerExecutionChain) handler : new HandlerExecutionChain(handler));

    String lookupPath = this.urlPathHelper.getLookupPathForRequest(request);
    // 迭代添加拦截器
    for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
        // 如果拦截器是MappedInterceptor，判断是否对该handler进行拦截，是的情况下添加
        if (interceptor instanceof MappedInterceptor) {
            MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
            if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
                chain.addInterceptor(mappedInterceptor.getInterceptor());
            }
        } else { // HandlerInterceptor直接添加，即通过HandingMapping属性配置的拦截器
            chain.addInterceptor(interceptor);
        }
    }
    return chain;
}
```

##### RequestMappingHandlerMapping

处理注解@RequestMapping及@Controller

- 实现InitializingBean接口，增加了bean初始化的能力，也就是说在bean初始化时可以做一些控制
- 实现EmbeddedValueResolverAware接口，即增加了读取属性文件的能力

继承自AbstractHandlerMethodMapping

```
//RequestMappingHandlerMapping
@Override
public void afterPropertiesSet() {
    this.config = new RequestMappingInfo.BuilderConfiguration();
    this.config.setUrlPathHelper(getUrlPathHelper());
    this.config.setPathMatcher(getPathMatcher());
    this.config.setSuffixPatternMatch(this.useSuffixPatternMatch);
    this.config.setTrailingSlashMatch(this.useTrailingSlashMatch);
    this.config.setRegisteredSuffixPatternMatch(this.useRegisteredSuffixPatternMatch);
    this.config.setContentNegotiationManager(getContentNegotiationManager());

    super.afterPropertiesSet();
}

//AbstractHandlerMethodMapping
@Override
public void afterPropertiesSet() {
    initHandlerMethods();
}
protected void initHandlerMethods() {
    //获取上下文中所有bean的name，不包含父容器
    for (String beanName : getCandidateBeanNames()) {
        if (!beanName.startsWith(SCOPED_TARGET_NAME_PREFIX)) {
            processCandidateBean(beanName);
        }
    }
    //日志记录HandlerMethods的总数量
    handlerMethodsInitialized(getHandlerMethods());
}


protected void processCandidateBean(String beanName) {
    Class<?> beanType = null;
    try {
        //根据name找出bean的类型
        beanType = obtainApplicationContext().getType(beanName);
    } catch (Throwable ex) {
        if (logger.isTraceEnabled()) {
            logger.trace("Could not resolve type for bean '" + beanName + "'", ex);
        }
    }
    //处理Controller和RequestMapping
    if (beanType != null && isHandler(beanType)) {
        detectHandlerMethods(beanName);
    }
}

//RequestMappingHandlerMapping
@Override
protected boolean isHandler(Class<?> beanType) {
    //获取@Controller和@RequestMapping
    return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
            AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}

//整个controller类的解析过程 
protected void detectHandlerMethods(Object handler) {
    //根据name找出bean的类型
    Class<?> handlerType = (handler instanceof String ?
                obtainApplicationContext().getType((String) handler) : handler.getClass());

    if (handlerType != null) {
            //获取真实的controller，如果是代理类获取父类
        Class<?> userType = ClassUtils.getUserClass(handlerType);
            //对真实的controller所有的方法进行解析和处理  key为方法对象，T为注解封装后的对象RequestMappingInfo
        Map<Method, T> methods = MethodIntrospector.selectMethods(userType,
                (MethodIntrospector.MetadataLookup<T>) method -> {
                        try {
                            // 调用子类RequestMappingHandlerMapping的getMappingForMethod方法进行处理，即根据RequestMapping注解信息创建匹配条件RequestMappingInfo对象
                            return getMappingForMethod(method, userType);
                        }catch (Throwable ex) {
                            throw new IllegalStateException("Invalid mapping on handler class [" +
                                    userType.getName() + "]: " + method, ex);
                        }
                    });
        if (logger.isTraceEnabled()) {
            logger.trace(formatMappings(userType, methods));
        }
        methods.forEach((method, mapping) -> {
                //找出controller中可外部调用的方法
            Method invocableMethod = AopUtils.selectInvocableMethod(method, userType);
                //注册处理方法
                registerHandlerMethod(handler, invocableMethod, mapping);
        });
    }
}

//requestMapping封装成RequestMappingInfo对象
protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
    RequestMappingInfo info = createRequestMappingInfo(method);//解析方法上的requestMapping
    if (info != null) {
        //解析方法所在类上的requestMapping
        RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);
        if (typeInfo != null) {
            info = typeInfo.combine(info);//合并类和方法上的路径,比如Controller类上有@RequestMapping("/demo")，方法的@RequestMapping("/demo1")，结果为"/demo/demo1"
        }
        String prefix = getPathPrefix(handlerType);//合并前缀
        if (prefix != null) {
            info = RequestMappingInfo.paths(prefix).options(this.config).build().combine(info);
        }
    }
    return info;
}

private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {
    //找到方法上的RequestMapping注解
    RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(element,                                                                         RequestMapping.class);
    //获取自定义的类型条件(自定义的RequestMapping注解)
    RequestCondition<?> condition = (element instanceof Class ?
                                            getCustomTypeCondition((Class<?>) element) :                                                                  getCustomMethodCondition((Method) element));
    return (requestMapping != null ? createRequestMappingInfo(requestMapping, condition) : null);
}

protected RequestMappingInfo createRequestMappingInfo(
    RequestMapping requestMapping, @Nullable RequestCondition<?> customCondition) {

    //获取RequestMapping注解的属性,封装成RequestMappingInfo对象
    RequestMappingInfo.Builder builder = RequestMappingInfo
        .paths(resolveEmbeddedValuesInPatterns(requestMapping.path()))
        .methods(requestMapping.method())
        .params(requestMapping.params())
        .headers(requestMapping.headers())
        .consumes(requestMapping.consumes())
        .produces(requestMapping.produces())
        .mappingName(requestMapping.name());
    if (customCondition != null) {
        builder.customCondition(customCondition);
    }
    return builder.options(this.config).build();
}

//再次封装成对应的对象    面向对象编程  每一个属性都存在多个值得情况需要排重封装
@Override
public RequestMappingInfo build() {
    ContentNegotiationManager manager = this.options.getContentNegotiationManager();

    PatternsRequestCondition patternsCondition = new PatternsRequestCondition(
        this.paths, this.options.getUrlPathHelper(), this.options.getPathMatcher(),
        this.options.useSuffixPatternMatch(), this.options.useTrailingSlashMatch(),
        this.options.getFileExtensions());

    return new RequestMappingInfo(this.mappingName, patternsCondition,
                                  new RequestMethodsRequestCondition(this.methods),
                                  new ParamsRequestCondition(this.params),
                                  new HeadersRequestCondition(this.headers),
                                  new ConsumesRequestCondition(this.consumes, this.headers),
                                  new ProducesRequestCondition(this.produces, this.headers, manager),
                                  this.customCondition);
}
protected void registerHandlerMethod(Object handler, Method method, T mapping) {
   this.mappingRegistry.register(mapping, handler, method);
}
//mapping是RequestMappingInfo对象   handler是controller类的beanName   method为接口方法
public void register(T mapping, Object handler, Method method) {
    ...
    this.readWriteLock.writeLock().lock();
    try {
        //beanName和method封装成HandlerMethod对象
        HandlerMethod handlerMethod = createHandlerMethod(handler, method);
        //验证RequestMappingInfo是否有对应不同的method，有则抛出异常
        validateMethodMapping(handlerMethod, mapping);
        //RequestMappingInfo和handlerMethod绑定
        this.mappingLookup.put(mapping, handlerMethod);

        List<String> directUrls = getDirectUrls(mapping);//可以配置多个url
        for (String url : directUrls) {
            //url和RequestMappingInfo绑定   可以根据url找到RequestMappingInfo，再找到handlerMethod
            this.urlLookup.add(url, mapping);
        }

        String name = null;
        if (getNamingStrategy() != null) {
            name = getNamingStrategy().getName(handlerMethod, mapping);
            addMappingName(name, handlerMethod);//方法名和Method绑定
        }

        CorsConfiguration corsConfig = initCorsConfiguration(handler, method, mapping);
        if (corsConfig != null) {
            this.corsLookup.put(handlerMethod, corsConfig);
        }
        //将RequestMappingInfo  url  handlerMethod绑定到MappingRegistration对象  放入map
        this.registry.put(mapping, new MappingRegistration<>(mapping, handlerMethod, directUrls, name));
    }
    finally {
        this.readWriteLock.writeLock().unlock();
    }
}
```

##### BeanNameUrlHandlerMapping

处理场景，id必须以/开头

```
public class HelloController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest request,
                                      HttpServletResponse response) throws Exception {
        ModelAndView mav = new ModelAndView("index");
        mav.addObject("message", "默认的映射处理器示例");
        return mav;
    }
<bean id="resourceView"
    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
<bean id="/name.do" class="com.luban.mvc.controller.HelloController"></bean>
//向上继承自ApplicationObjectSupport实现ApplicationContextAware接口
public abstract class ApplicationObjectSupport implements ApplicationContextAware {
    protected final Log logger = LogFactory.getLog(this.getClass());
    @Nullable
    private ApplicationContext applicationContext;
    ...

    public final void setApplicationContext(@Nullable ApplicationContext context) throws BeansException {
        if (context == null && !this.isContextRequired()) {
            this.applicationContext = null;
            this.messageSourceAccessor = null;
        } else if (this.applicationContext == null) {
            if (!this.requiredContextClass().isInstance(context)) {
                throw new ApplicationContextException("Invalid application context: needs to be of type [" + this.requiredContextClass().getName() + "]");
            }

            this.applicationContext = context;
            this.messageSourceAccessor = new MessageSourceAccessor(context);
            this.initApplicationContext(context);//模板方法，调子类
        } else if (this.applicationContext != context) {
            throw new ApplicationContextException("Cannot reinitialize with different application context: current one is [" + this.applicationContext + "], passed-in one is [" + context + "]");
        }

    }
    ...
}

@Override
public void initApplicationContext() throws ApplicationContextException {
    super.initApplicationContext();//加载拦截器
    // 处理url和bean name，具体注册调用父类AbstractUrlHandlerMapping类完成
    detectHandlers();
}

//建立当前ApplicationContext中controller和url的对应关系
protected void detectHandlers() throws BeansException {
    // 获取应用上下文
    ApplicationContext applicationContext = obtainApplicationContext();
    //获取ApplicationContext中的所有bean的name(也是id，即@Controller的属性值)
    String[] beanNames = (this.detectHandlersInAncestorContexts ?
                          BeanFactoryUtils.beanNamesForTypeIncludingAncestors(applicationContext,                                      Object.class) : applicationContext.getBeanNamesForType(Object.class));

    //遍历所有beanName
    for (String beanName : beanNames) {
        // 通过模板方法模式调用BeanNameUrlHandlerMapping子类处理
        String[] urls = determineUrlsForHandler(beanName);//判断是否以/开始
        if (!ObjectUtils.isEmpty(urls)) {
            //调用父类AbstractUrlHandlerMapping将url与handler存入map
            registerHandler(urls, beanName);
        }
    }
}

protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
    Assert.notNull(urlPath, "URL path must not be null");
    Assert.notNull(handler, "Handler object must not be null");
    Object resolvedHandler = handler;

    // Eagerly resolve handler if referencing singleton via name.
    if (!this.lazyInitHandlers && handler instanceof String) {
        String handlerName = (String) handler;
        ApplicationContext applicationContext = obtainApplicationContext();
        if (applicationContext.isSingleton(handlerName)) {
            resolvedHandler = applicationContext.getBean(handlerName);
        }
    }
    //已存在且指向不同的Handler抛异常
    Object mappedHandler = this.handlerMap.get(urlPath);
    if (mappedHandler != null) {
        if (mappedHandler != resolvedHandler) {
            throw new IllegalStateException(
                "Cannot map " + getHandlerDescription(handler) + " to URL path [" + urlPath +
                "]: There is already " + getHandlerDescription(mappedHandler) + " mapped.");
        }
    }
    else {
        if (urlPath.equals("/")) {
            if (logger.isTraceEnabled()) {
                logger.trace("Root mapping to " + getHandlerDescription(handler));
            }
            setRootHandler(resolvedHandler);//设置根处理器，即请求/的时候
        }
        else if (urlPath.equals("/*")) {
            if (logger.isTraceEnabled()) {
                logger.trace("Default mapping to " + getHandlerDescription(handler));
            }
            setDefaultHandler(resolvedHandler);//默认处理器
        }
        else {
            this.handlerMap.put(urlPath, resolvedHandler);//注册进map
            if (logger.isTraceEnabled()) {
                logger.trace("Mapped [" + urlPath + "] onto " + getHandlerDescription(handler));
            }
        }
    }
}
```

- 处理器bean的id/name为一个url请求路径，前面有"/"；
- 如果多个url映射同一个处理器bean，那么就需要定义多个bean，导致容器创建多个处理器实例，占用内存空间；
- 处理器bean定义与url请求耦合在一起。

\##### SimpleUrlHandlerMapping  

间接实现了org.springframework.web.servlet.HandlerMapping接口，直接实现该接口的是org.springframework.web.servlet.handler.AbstractHandlerMapping抽象类，映射Url与请求handler bean。支持映射bean实例和映射bean名称

```
public class SimpleUrlHandlerMapping extends AbstractUrlHandlerMapping {
    // 存储url和bean映射
    private final Map<String, Object> urlMap = new LinkedHashMap<>();
    // 注入property的name为mappings映射
    public void setMappings(Properties mappings) {
        CollectionUtils.mergePropertiesIntoMap(mappings, this.urlMap);
    }
    // 注入property的name为urlMap映射
    public void setUrlMap(Map<String, ?> urlMap) {
        this.urlMap.putAll(urlMap);
    }
    public Map<String, ?> getUrlMap() {
        return this.urlMap;
    }
    // 实例化本类实例入口
    @Override
    public void initApplicationContext() throws BeansException {
        // 调用父类AbstractHandlerMapping的initApplicationContext方法，只要完成拦截器的注册
        super.initApplicationContext();
        // 处理url和bean name，具体注册调用父类完成
        registerHandlers(this.urlMap);
    }
    // 注册映射关系，及将property中的值解析到map对象中，key为url，value为bean id或name
    protected void registerHandlers(Map<String, Object> urlMap) throws BeansException {
        if (urlMap.isEmpty()) {
            logger.warn("Neither 'urlMap' nor 'mappings' set on SimpleUrlHandlerMapping");
        } else {
            urlMap.forEach((url, handler) -> {
                // 增加以"/"开头
                if (!url.startsWith("/")) {
                    url = "/" + url;
                }
                // 去除handler bean名称的空格
                if (handler instanceof String) {
                    handler = ((String) handler).trim();
                }
                // 调用父类AbstractUrlHandlerMapping完成映射
                registerHandler(url, handler);
            });
        }
    }

}
```

SimpleUrlHandlerMapping类主要接收用户设定的url与handler的映射关系，其实际的工作都是交由其父类来完成的

- AbstractHandlerMapping
  在创建初始化SimpleUrlHandlerMapping类时，调用其父类的initApplicationContext()方法，该方法完成拦截器的初始化

```
@Override
protected void initApplicationContext() throws BeansException {
    // 空实现。子类可重写此方法以注册额外的拦截器
    extendInterceptors(this.interceptors);
    // 从上下文中查询拦截器并添加到拦截器列表中
    detectMappedInterceptors(this.adaptedInterceptors);
    // 初始化拦截器
    initInterceptors();
}

// 查找实现了MappedInterceptor接口的bean，并添加到映射拦截器列表
protected void detectMappedInterceptors(List<HandlerInterceptor> mappedInterceptors) {
    mappedInterceptors.addAll(BeanFactoryUtils.beansOfTypeIncludingAncestors(
                    obtainApplicationContext(), MappedInterceptor.class, true, false).values());
}

// 将自定义bean设置到适配拦截器中，bean需实现HandlerInterceptor或WebRequestInterceptor
protected void initInterceptors() {
    if (!this.interceptors.isEmpty()) {
        for (int i = 0; i < this.interceptors.size(); i++) {
            Object interceptor = this.interceptors.get(i);
            if (interceptor == null) {
                throw new IllegalArgumentException("Entry number " + i + " in interceptors array is null");
            }
            this.adaptedInterceptors.add(adaptInterceptor(interceptor));
        }
    }
}
```

- AbstractUrlHandlerMapping

在创建初始化SimpleUrlHandlerMapping类时，调用AbstractUrlHandlerMapping类的registerHandler(urlPath,handler)方法

```java
protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
    Assert.notNull(urlPath, "URL path must not be null");
    Assert.notNull(handler, "Handler object must not be null");
    Object resolvedHandler = handler;

    // 不是懒加载，默认为false，即不是，通过配置SimpleUrlHandlerMapping属性lazyInitHandlers的值进行控制
    // 如果不是懒加载并且handler为单例，即从上下文中查询实例处理，此时resolvedHandler为handler实例对象；
    // 如果是懒加载或者handler不是单例，即resolvedHandler为handler逻辑名
    if (!this.lazyInitHandlers && handler instanceof String) {
        String handlerName = (String) handler;
        ApplicationContext applicationContext = obtainApplicationContext();
        // 如果handler是单例，通过bean的scope控制
        if (applicationContext.isSingleton(handlerName)) {
            resolvedHandler = applicationContext.getBean(handlerName);
        }
    }

    Object mappedHandler = this.handlerMap.get(urlPath);
    if (mappedHandler != null) {
        if (mappedHandler != resolvedHandler) {
            throw new IllegalStateException(
                    "Cannot map " + getHandlerDescription(handler) + " to URL path [" + urlPath +
                    "]: There is already " + getHandlerDescription(mappedHandler) + " mapped.");
        }
    }
    else {
        if (urlPath.equals("/")) {
            if (logger.isInfoEnabled()) {
                logger.info("Root mapping to " + getHandlerDescription(handler));
            }
            setRootHandler(resolvedHandler);
        }
        else if (urlPath.equals("/*")) {
            if (logger.isInfoEnabled()) {
                logger.info("Default mapping to " + getHandlerDescription(handler));
            }
            setDefaultHandler(resolvedHandler);
        }
        else {
            // 把url与handler（名称或实例）放入map，以供后续使用
            this.handlerMap.put(urlPath, resolvedHandler);
            if (logger.isInfoEnabled()) {
                logger.info("Mapped URL path [" + urlPath + "] onto " + getHandlerDescription(handler));
            }
        }
    }
}
```

## Filter和拦截器的区别

Fileter 需要配置在web.xml,进入servlet之前就会触发,过滤器拦截的dispatcherServlet

![image-20201202222212005](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201202222212005.png)

拦截器更细

拦截前 method 权限拦截

拦截中

拦截后 规范来说必须执行一次





## 请求处理

```java
//Servlet里面的经典方法，处理get，head，post，delete请求
protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                if (ifModifiedSince < lastModified) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
            
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);
            
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
            
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
            
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
            
        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
            
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
```

### FrameworkServlet

```java
//刚才所有的dopost，doget这些方法，都在子类里面重写了，统一去processRequest
@Override
	protected final void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
	}
```

```java
/**
	 * Process this request, publishing an event regardless of the outcome.
	 * <p>The actual event handling is performed by the abstract
	 * {@link #doService} template method.
	 */
	protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;

		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		LocaleContext localeContext = buildLocaleContext(request);

		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    //整合请求的属性，执行的线程
		initContextHolders(request, localeContext, requestAttributes);

		try {
      //然后去处理请求
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}
```

## DispatcherServlet

```java
/**
	 * Exposes the DispatcherServlet-specific request attributes and delegates to {@link #doDispatch}
	 * for the actual dispatching.
	 */
	@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    //记录日志
		logRequest(request);

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			attributesSnapshot = new HashMap<>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// Make framework objects available to handlers and view objects.
    //这个地方就是看你配置了什么模版信息，主题
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		if (this.flashMapManager != null) {
			FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
			if (inputFlashMap != null) {
				request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
			}
			request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
			request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
		}

		try {
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
		}
	}
```

```java
/**
	 * Process the actual dispatching to the handler.
	 * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
	 * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
	 * to find the first that supports the handler class.
	 * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
	 * themselves to decide which methods are acceptable.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @throws Exception in case of any kind of processing failure
	 */
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
        //主要是这个地方就要开始查询适配器，然后去解析了
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
        //查询适配器，看看是否根之前查出来的处理器适配
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
        
				//前拦截
        // 可以如果配置了就按照配置的来
        // 如果没有配置就用默认的
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
        // 处理请求
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}
				
        // 注入默认视图
				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```

```java
/**
	 * Return the HandlerExecutionChain for this request.
	 * <p>Tries all handler mappings in order.
	 * @param request current HTTP request
	 * @return the HandlerExecutionChain, or {@code null} if no handler could be found
	 */
	@Nullable
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```

## 初始化拦截器，处理器

```java
/**
	 * Look up a handler for the given request, falling back to the default
	 * handler if no specific one is found.
	 * @param request current HTTP request
	 * @return the corresponding handler instance, or the default handler
	 * @see #getHandlerInternal
	 */
	@Override
	@Nullable
	public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		Object handler = getHandlerInternal(request);
		if (handler == null) {
			handler = getDefaultHandler();
		}
		if (handler == null) {
			return null;
		}
		// Bean name or resolved handler?
		if (handler instanceof String) {
			String handlerName = (String) handler;
			handler = obtainApplicationContext().getBean(handlerName);
		}
		//这里面会整合handler和HandlerExecution
		HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);

		if (logger.isTraceEnabled()) {
			logger.trace("Mapped to " + handler);
		}
		else if (logger.isDebugEnabled() && !request.getDispatcherType().equals(DispatcherType.ASYNC)) {
			logger.debug("Mapped to " + executionChain.getHandler());
		}

		if (hasCorsConfigurationSource(handler) || CorsUtils.isPreFlightRequest(request)) {
			CorsConfiguration config = (this.corsConfigurationSource != null ? this.corsConfigurationSource.getCorsConfiguration(request) : null);
			CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
			config = (config != null ? config.combine(handlerConfig) : handlerConfig);
			executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
		}

		return executionChain;
	}
```



## 初始化拦截器

```java
/**
	 * Create a new HandlerExecutionChain.
	 * @param handler the handler object to execute
	 * @param interceptors the array of interceptors to apply
	 * (in the given order) before the handler itself executes
	 */
	public HandlerExecutionChain(Object handler, @Nullable HandlerInterceptor... interceptors) {
		if (handler instanceof HandlerExecutionChain) {
			HandlerExecutionChain originalChain = (HandlerExecutionChain) handler;
			this.handler = originalChain.getHandler();
			this.interceptorList = new ArrayList<>();
			CollectionUtils.mergeArrayIntoCollection(originalChain.getInterceptors(), this.interceptorList);
			CollectionUtils.mergeArrayIntoCollection(interceptors, this.interceptorList);
		}
		else {
			this.handler = handler;
			this.interceptors = interceptors;
		}
	}
```

