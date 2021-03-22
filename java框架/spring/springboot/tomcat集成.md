# 零配置及内嵌tomcat原理

## 零配置原理

基于spring新特性javaConfig

Spring JavaConfig是Spring社区的产品，使用java代码配置Spring IoC容器。不需要使用XML配置。

JavaConfig的优点：

- 面向对象的配置。配置被定义为JavaConfig类，因此用户可以充分利用Java中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean方法等。
- 减少或消除XML配置。许多开发人员不希望在XML和Java之间来回切换。JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。
- 类型安全和重构友好。JavaConfig提供了一种类型安全的方法来配置Spring容器。由于Java 5.0对泛型的支持，现在可以按类型而不是按名称检索bean，不需要任何强制转换或基于字符串的查找

web.xml：

ContextLoaderListener：监听servlet启动、销毁，初始化ioc完成依赖注入

DispatcherServlet：接收tomcat解析之后的http请求，匹配controller处理业务

代码替换：

```java
AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
ac.register(Start.class);
//ac.refresh();

DispatcherServlet servlet = new DispatcherServlet(ac);
```

applicationContext.xml:

component-scan：扫描   注解替换： @ComponentScan

<beans>  <bean>  注解替换：  @Configuration +@Bean  

springmvc.xml:

component-scan：扫描  只扫描@Controller   注解替换： @ComponentScan

视图解析器，json转换，国际化，编码。。。。  注解替换：  @Configuration +@Bean  

代码替换：

```java
@Configuration
@EnableWebMvc
public class MyConfig implements WebMvcConfigurer{

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter builder = new FastJsonHttpMessageConverter();
        converters.add(builder);
    }
}
```

## 内嵌tomcat原理

apache 在tomcat7提供内嵌版本的tomcat  tomcat.jar

```java
public static void createServer() throws Exception{
    Tomcat tomcat = new Tomcat();
    tomcat.setPort(8088);

    tomcat.addWebapp("/","D://tomcat");//完成监听

    tomcat.start();
    tomcat.getServer().await();
}
```

### 1、addWebapp解析

通过反射加载监听类LifecycleListener：ContextConfig

ContextConfig监听方法进行配置启动：configureStart()

  webConfig()：读取/WEB-INF/web.xml（如果有）

   processServletContainerInitializers()：完成spi类的扫描和加载（ServletContainerInitializer），存入initializerClassMap

​     遍历initializerClassMap将ServletContainerInitializer存入context（StandardContext的全局变量initializers）

### 2、start解析

```java
public void start() throws LifecycleException {
    getServer();//初始化server容器及service子容器
    getConnector();//初始化Connector
    server.start();//调用startInternal()
}
```

StandardContext的startInternal()方法：遍历initializers，调用ServletContainerInitializer的onStartup()方法

```java
for (Map.Entry<ServletContainerInitializer, Set<Class<?>>> entry :
                initializers.entrySet()) {
    try {
        entry.getKey().onStartup(entry.getValue(),
                                 getServletContext());
    } catch (ServletException e) {
        log.error(sm.getString("standardContext.sciFail"), e);
        ok = false;
        break;
    }
}
```

1、加载监听器LifecycleListener

2、监听器实现类：ContextConfig->lifecycleEvent   ->webConfig

#### 这里面实现了spi机制

```java
protected void webConfig() {
        /*
         * Anything and everything can override the global and host defaults.
         * This is implemented in two parts
         * - Handle as a web fragment that gets added after everything else so
         *   everything else takes priority
         * - Mark Servlets as overridable so SCI configuration can replace
         *   configuration from the defaults
         */

        /*
         * The rules for annotation scanning are not as clear-cut as one might
         * think. Tomcat implements the following process:
         * - As per SRV.1.6.2, Tomcat will scan for annotations regardless of
         *   which Servlet spec version is declared in web.xml. The EG has
         *   confirmed this is the expected behaviour.
         * - As per http://java.net/jira/browse/SERVLET_SPEC-36, if the main
         *   web.xml is marked as metadata-complete, JARs are still processed
         *   for SCIs.
         * - If metadata-complete=true and an absolute ordering is specified,
         *   JARs excluded from the ordering are also excluded from the SCI
         *   processing.
         * - If an SCI has a @HandlesType annotation then all classes (except
         *   those in JARs excluded from an absolute ordering) need to be
         *   scanned to check if they match.
         */
        WebXmlParser webXmlParser = new WebXmlParser(context.getXmlNamespaceAware(),
                context.getXmlValidation(), context.getXmlBlockExternal());

        Set<WebXml> defaults = new HashSet<>();
        defaults.add(getDefaultWebXmlFragment(webXmlParser));

        WebXml webXml = createWebXml();

        // Parse context level web.xml
  			// 读取web.xml配置文件
        InputSource contextWebXml = getContextWebXmlSource();
        if (!webXmlParser.parseWebXml(contextWebXml, webXml, false)) {
            ok = false;
        }
				
  			// 
        ServletContext sContext = context.getServletContext();

        // Ordering is important here

        // Step 1. Identify all the JARs packaged with the application and those
        // provided by the container. If any of the application JARs have a
        // web-fragment.xml it will be parsed at this point. web-fragment.xml
        // files are ignored for container provided JARs.
        Map<String,WebXml> fragments = processJarsForWebFragments(webXml, webXmlParser);

        // Step 2. Order the fragments.
        Set<WebXml> orderedFragments = null;
        orderedFragments =
                WebXml.orderWebFragments(webXml, fragments, sContext);

        // Step 3. Look for ServletContainerInitializer implementations
  			// 这个地方就是spi的方法
        if (ok) {
            processServletContainerInitializers();
        }

        if  (!webXml.isMetadataComplete() || typeInitializerMap.size() > 0) {
            // Step 4. Process /WEB-INF/classes for annotations and
            // @HandlesTypes matches
            Map<String,JavaClassCacheEntry> javaClassCache = new HashMap<>();

            if (ok) {
                WebResource[] webResources =
                        context.getResources().listResources("/WEB-INF/classes");

                for (WebResource webResource : webResources) {
                    // Skip the META-INF directory from any JARs that have been
                    // expanded in to WEB-INF/classes (sometimes IDEs do this).
                    if ("META-INF".equals(webResource.getName())) {
                        continue;
                    }
                    // 这个地方里面就用了asm技术，扫描class文件
                    // 最后通过反射实例化对象
                    processAnnotationsWebResource(webResource, webXml,
                            webXml.isMetadataComplete(), javaClassCache);
                }
            }

            // Step 5. Process JARs for annotations and
            // @HandlesTypes matches - only need to process those fragments we
            // are going to use (remember orderedFragments includes any
            // container fragments)
            if (ok) {
                processAnnotations(
                        orderedFragments, webXml.isMetadataComplete(), javaClassCache);
            }

            // Cache, if used, is no longer required so clear it
            javaClassCache.clear();
        }

        if (!webXml.isMetadataComplete()) {
            // Step 6. Merge web-fragment.xml files into the main web.xml
            // file.
            if (ok) {
                ok = webXml.merge(orderedFragments);
            }

            // Step 7. Apply global defaults
            // Have to merge defaults before JSP conversion since defaults
            // provide JSP servlet definition.
            webXml.merge(defaults);

            // Step 8. Convert explicitly mentioned jsps to servlets
            if (ok) {
                convertJsps(webXml);
            }

            // Step 9. Apply merged web.xml to Context
            if (ok) {
                configureContext(webXml);
            }
        } else {
            webXml.merge(defaults);
            convertJsps(webXml);
            configureContext(webXml);
        }

        if (context.getLogEffectiveWebXml()) {
            log.info("web.xml:\n" + webXml.toXml());
        }

        // Always need to look for static resources
        // Step 10. Look for static resources packaged in JARs
        if (ok) {
            // Spec does not define an order.
            // Use ordered JARs followed by remaining JARs
            Set<WebXml> resourceJars = new LinkedHashSet<>();
            for (WebXml fragment : orderedFragments) {
                resourceJars.add(fragment);
            }
            for (WebXml fragment : fragments.values()) {
                if (!resourceJars.contains(fragment)) {
                    resourceJars.add(fragment);
                }
            }
            processResourceJARs(resourceJars);
            // See also StandardContext.resourcesStart() for
            // WEB-INF/classes/META-INF/resources configuration
        }

        // Step 11. Apply the ServletContainerInitializer config to the
        // context
      
        if (ok) {
            // initializerClassMap 第三部的时候就已经初始化好的class + 实现类的map遍历
            // 加到上下文里面
            for (Map.Entry<ServletContainerInitializer,
                    Set<Class<?>>> entry :
                        initializerClassMap.entrySet()) {
                if (entry.getValue().isEmpty()) {
                    context.addServletContainerInitializer(
                            entry.getKey(), null);
                } else {
                    context.addServletContainerInitializer(
                            entry.getKey(), entry.getValue());
                }
            }
        }
    }
```

### spi内部代码

```java
protected void processServletContainerInitializers() {

        List<ServletContainerInitializer> detectedScis;
        try {
          	// 这个地方就是new一个classloader
            WebappServiceLoader<ServletContainerInitializer> loader = new WebappServiceLoader<>(context);
          	// 加载启动类
            detectedScis = loader.load(ServletContainerInitializer.class);
        } catch (IOException e) {
            log.error(sm.getString(
                    "contextConfig.servletContainerInitializerFail",
                    context.getName()),
                e);
            ok = false;
            return;
        }

        for (ServletContainerInitializer sci : detectedScis) {
            initializerClassMap.put(sci, new HashSet<Class<?>>());

            HandlesTypes ht;
            try {
                // 拿到带HandlesTypes 这个注解的类
                // springboot启动的时候 SpringServletContainerInitializer这个类就会被找出来
                ht = sci.getClass().getAnnotation(HandlesTypes.class);
            } catch (Exception e) {
                if (log.isDebugEnabled()) {
                    log.info(sm.getString("contextConfig.sci.debug",
                            sci.getClass().getName()),
                            e);
                } else {
                    log.info(sm.getString("contextConfig.sci.info",
                            sci.getClass().getName()));
                }
                continue;
            }
            if (ht == null) {
                continue;
            }
             // 这个地方会拿注解的value，其实就是一个class
            Class<?>[] types = ht.value();
            if (types == null) {
                continue;
            }

            // 若配置了多个class
            for (Class<?> type : types) {
                if (type.isAnnotation()) {
                    handlesTypesAnnotations = true;
                } else {
                    handlesTypesNonAnnotations = true;
                }
                // 获取实现类
                Set<ServletContainerInitializer> scis =
                        typeInitializerMap.get(type);
                if (scis == null) {
                    scis = new HashSet<>();
                    // WebApplicationInitializer.class 会作为key往里面插入
                    // 实现类作为value
                    typeInitializerMap.put(type, scis);
                }
                scis.add(sci);
            }
        }
    }
```



3、自定义类加载器加载ServletContainerInitializer实现类  扫描META-INF/services/  放入 initializerClassMap

4、initializerClassMap转换到StandardContext的全局变量initializers

5、tomcat启动start方法调用StandardContext.startInternal  遍历initializers  调用onstartup方法



![image-20201203231321005](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201203231321005.png)

## SpringApplication的run方法

```java
/**
	 * Run the Spring application, creating and refreshing a new
	 * {@link ApplicationContext}.
	 * @param args the application arguments (usually passed from a Java main method)
	 * @return a running {@link ApplicationContext}
	 */
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
      // 会之前创建一个容器，会判断环境是servlet还是reactive
      // servlet就是创建AnnotationConfigServletWebServerApplicationContext 这个上下文
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			// 启动内嵌tomcat的方法也在这里面
      // 这里面其实就是调用application.refresh()方法，回ioc了
      // refresh()里面有一个onrefresh()方法给子类实现的
      // AnnotationConfigServletWebServerApplicationContext 这个容器的父类实现了这个方法
      // onrefresh会调用会调用 createWebServer()创建一个web服务器
      refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

###  ServletWebServerApplicationContext

```java
// 这里判断是因为她不一定用的tomcat，有可能是jetty，undertow
private void createWebServer() {
		WebServer webServer = this.webServer;
		ServletContext servletContext = getServletContext();
		if (webServer == null && servletContext == null) {
      // 就是单例池里面找bean，bytype
      //  他会是这个类TomcatServletWebServerFactory
      // 这个类通过spring的spi加载进来的
			ServletWebServerFactory factory = getWebServerFactory();
      //拿出来之后因为默认是Tomcat，
			this.webServer = factory.getWebServer(getSelfInitializer());
		}
   // war包运行
		else if (servletContext != null) {
			try {
				getSelfInitializer().onStartup(servletContext);
			}
			catch (ServletException ex) {
				throw new ApplicationContextException("Cannot initialize servlet context", ex);
			}
		}
		initPropertySources();
	}
```

## TomcatServletWebServerFactory

```java
@Override
	public WebServer getWebServer(ServletContextInitializer... initializers) {
		if (this.disableMBeanRegistry) {
			Registry.disableRegistry();
		}
    // tomcat经典的启动方法，new出来，配置各种功能
		Tomcat tomcat = new Tomcat();
		File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
		tomcat.setBaseDir(baseDir.getAbsolutePath());
		Connector connector = new Connector(this.protocol);
		connector.setThrowOnFailure(true);
		tomcat.getService().addConnector(connector);
		customizeConnector(connector);
		tomcat.setConnector(connector);
		tomcat.getHost().setAutoDeploy(false);
		configureEngine(tomcat.getEngine());
		for (Connector additionalConnector : this.additionalTomcatConnectors) {
			tomcat.getService().addConnector(additionalConnector);
		}
		prepareContext(tomcat.getHost(), initializers);
   	// 这里面左右回去掉initialize(), tomcat.start()启动
		return getTomcatWebServer(tomcat);
	}
```

