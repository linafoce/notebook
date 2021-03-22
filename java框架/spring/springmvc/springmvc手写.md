# 【11.17】课堂笔记

springmvc原理解析--手写模拟springmvc  demo

1、web框架功能分析

2、模拟springmvc写web框架

3、springmvc整体架构

servlet和spring  实现web框架

IOC

处理器：做业务处理的组件  类  方法  跟url有映射关系

功能分析：

​	使用内嵌tomcat作为servlet容器，

​	打开IOC扫描，后置处理器，IOC取出处理器   处理器

 

​	spring注解：@Controller  普通的bean   Map<url，Object处理器>

 

​	springmvc注解：@RequestMapping  url

 

## 处理器执行

1. @Controller   @RequestMapping(url)方法   反射执行   <url,method>  反射

2. servlet  service()  映射路径  /  if  else

3.  Controller  handleRequest   (Controller)处理器 调用 handleRequest

4.  HttpRequestHandler  handleRequest  (HttpRequestHandler)处理器 调用 handleRequest

 

 

​	**/beanName  --->  bean处理器  </id,处理器>**

 

 

## 适配器：跟映射器一一对应    

**处理器是由适配器执行的 url跟method的对应关系**

​	统一的接口  

​	判断适配是否成功   Object处理器  instanseOf  Controller

​	执行处理逻辑    (Controller)处理器 调用 handleRequest

 

json返回  User  序列化  @responseBody

参数绑定   @RequestParam（value）

不加注解：参数是怎么注入的

jdk7：asm

jdk8：可以通过反射获取参数名

```java
/**
*	 处理器接口
*/
public interface HandlerMapping {


    Object getHandlerMapping(String requestURI);
}

```

### beanName映射器

```java
@Component
public class BeanNameHandlerMapping  implements HandlerMapping,InstantiationAwareBeanPostProcessor {

    public static Map<String, Controller> map = new HashMap<>();

    /**
     * 实例化后的后置处理器，相当于把bean，beanName放到一个Map里面
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if(beanName.startsWith("/")){
            map.put(beanName,(Controller)bean);
        }
        return true;
    }

    /**
     * 获取处理器
     * @param requestURI
     * @return
     */
    @Override
    public Object getHandlerMapping(String requestURI) {
        return map.get(requestURI);
    }
}
```

### 注解映射器

```java
@Component
public class AnnotationHadlerMapping  implements HandlerMapping,InstantiationAwareBeanPostProcessor {

    public static Map<String,RequestMappingInfo> map = new HashMap<>();

    /**
     * 本质也是实现实例化后的后置处理器
     * 但是由于是注解，调用方法的时候需要通过反射调用方法，bean是个对象，也不是方法
     * 所以维护了一个RequestMappingInfo，把bean和方法都扔在里面
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    //bean里面会有多个处理器,说白了就是取所有带@RequestMapping的方法
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        Method[] methods = bean.getClass().getDeclaredMethods();
        for(Method method : methods){
            if(method.isAnnotationPresent(RequestMapping.class)) {
                RequestMappingInfo requestMappingInfo = createRequestMappingInfo(method, bean);
                map.put(requestMappingInfo.getUri(), requestMappingInfo);
            }
        }
        return true;
    }
    
    private RequestMappingInfo createRequestMappingInfo(Method method,Object bean) {
        RequestMappingInfo requestMappingInfo = new RequestMappingInfo();
        if(method.isAnnotationPresent(RequestMapping.class)){
            requestMappingInfo.setMethod(method);
            requestMappingInfo.setUri(method.getDeclaredAnnotation(RequestMapping.class).value());
            requestMappingInfo.setObj(bean);
        }
        return requestMappingInfo;
    }

    @Override
    public Object getHandlerMapping(String requestURI) {
        return map.get(requestURI);
    }


}
```

### 适配器

#### 根据beanName的适配器

```java
@Component
public class BeanNameHandlerAdapter implements HandlerAdapter{
    @Override
   //判断hanlde是否适配当前适配器
    public boolean supports(Object handler) {
        //判断是否是RequestMapping子类
        return (handler instanceof Controller);
    }

    @Override
    public Object handle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        return ((Controller)handler).handler(request, response);
    }
}
```

### 根据注解的适配器

```java
@Component
public class AnnotationHandlerAdapter implements HandlerAdapter{

    @Override
    public boolean supports(Object handler) {
        //判断是否是RequestMapping子类
        return (handler instanceof RequestMappingInfo);
    }

    @Override
    //参数绑定
    public Object handle(HttpServletRequest request, HttpServletResponse response, Object handler){
        RequestMappingInfo requestMappingInfo = (RequestMappingInfo)handler;
        Map<String, String[]> paramMap = request.getParameterMap();//请求携带的参数
        Method method = requestMappingInfo.getMethod();//方法定义的参数
        Parameter[] parameters = method.getParameters();

        Object[] params = new Object[method.getParameterTypes().length];
        for(int i=0; i<parameters.length; i++){
            for(Map.Entry<String, String[]> entry : paramMap.entrySet()){
                if(parameters[i].getAnnotation(RequestParam.class) != null && entry.getKey()!= null &&
                        entry.getKey().equals(parameters[i].getAnnotation(RequestParam.class).value())){
                    params[i] = entry.getValue()[0];
                //jdk1.8实现反射获取方法名   1.8之前使用asm实现
                }else if(entry.getKey().equals(parameters[i].getName())){
                    params[i] = entry.getValue()[0];
                }
            }

            //传入request和response
            if(ServletRequest.class.isAssignableFrom(parameters[i].getType())){
                params[i] = request;
            }else if(ServletResponse.class.isAssignableFrom(parameters[i].getType())){
                params[i] = response;
            }
        }

        try {
           //通过反射去执行方法，传入bean，params两个参数
            Object result = method.invoke(requestMappingInfo.getObj(),params);
            if (result instanceof String) {
                if ("forward".equals(((String) result).split(":")[0])) {
                    request.getRequestDispatcher(((String) result).split(":")[1]).forward(request, response);
                } else {
                    response.sendRedirect(((String) result).split(":")[1]);
                }
            }else{
                if(method.isAnnotationPresent(ResponseBody.class)){
                    return JSON.toJSONString(result);
                }
            }

            return result;
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```





国际化

异常

扩展

拦截器

过滤器

监听器

## SPI

### Tomcat  SPI：服务发现机制

特定的目录(目录路径是定死的，相对路径  META-INF/service +  接口全限定名)下：实现类(字符串)

```java

//SPI
//servlet3.0核心接口
public class Myinit implements ServletContainerInitializer{
		
  //重写启动方法
    @Override
    public void onStartup(Set<Class<?>> c,ServletContext servletContext) {
       //ioc初始化
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(Start.class);

        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic dy = servletContext.addServlet("app",servlet);
        dy.setLoadOnStartup(1);
        dy.addMapping("*.do");

      /*  JspServlet jspServlet = new JspServlet();
        ServletRegistration.Dynamic dy1 = servletContext.addServlet("app1",jspServlet);
        dy1.setLoadOnStartup(2);
        dy1.addMapping("*.jsp");*/
    }

}
```

```java
 /**
     * Scan JARs for ServletContainerInitializer implementations.
     扫描实现类
     */
    protected void processServletContainerInitializers() {

        List<ServletContainerInitializer> detectedScis;
        try {
            WebappServiceLoader<ServletContainerInitializer> loader = new WebappServiceLoader<>(context);
            //loader.load会获取类加载器
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
            Class<?>[] types = ht.value();
            if (types == null) {
                continue;
            }

            for (Class<?> type : types) {
                if (type.isAnnotation()) {
                    handlesTypesAnnotations = true;
                } else {
                    handlesTypesNonAnnotations = true;
                }
                Set<ServletContainerInitializer> scis =
                        typeInitializerMap.get(type);
                if (scis == null) {
                    scis = new HashSet<>();
                    typeInitializerMap.put(type, scis);
                }
                scis.add(sci);
            }
        }
    }
```

## Load方法

```java
/**
     * Load the providers for a service type.
     *
     * @param serviceType the type of service to load
     * @return an unmodifiable collection of service providers
     * @throws IOException if there was a problem loading any service
     */
    public List<T> load(Class<T> serviceType) throws IOException {
        //解析文件名，SERVICES写死的目录前缀
        String configFile = SERVICES + serviceType.getName();

        LinkedHashSet<String> applicationServicesFound = new LinkedHashSet<>();
        LinkedHashSet<String> containerServicesFound = new LinkedHashSet<>();
				
      	//获取classLoader
        ClassLoader loader = servletContext.getClassLoader();

        // if the ServletContext has ORDERED_LIBS, then use that to specify the
        // set of JARs from WEB-INF/lib that should be used for loading services
        @SuppressWarnings("unchecked")
        List<String> orderedLibs =
                (List<String>) servletContext.getAttribute(ServletContext.ORDERED_LIBS);
        if (orderedLibs != null) {
            // handle ordered libs directly, ...
            for (String lib : orderedLibs) {
                URL jarUrl = servletContext.getResource(LIB + lib);
                if (jarUrl == null) {
                    // should not happen, just ignore
                    continue;
                }

                String base = jarUrl.toExternalForm();
                URL url;
                if (base.endsWith("/")) {
                    url = new URL(base + configFile);
                } else {
                    url = JarFactory.getJarEntryURL(jarUrl, configFile);
                }
                try {
                    parseConfigFile(applicationServicesFound, url);
                } catch (FileNotFoundException e) {
                    // no provider file found, this is OK
                }
            }

            // and the parent ClassLoader for all others
            loader = context.getParentClassLoader();
        }

        Enumeration<URL> resources;
        if (loader == null) {
            resources = ClassLoader.getSystemResources(configFile);
        } else {
            resources = loader.getResources(configFile);
        }
        while (resources.hasMoreElements()) {
            parseConfigFile(containerServicesFound, resources.nextElement());
        }

        // Filter the discovered container SCIs if required
        if (containerSciFilterPattern != null) {
            Iterator<String> iter = containerServicesFound.iterator();
            while (iter.hasNext()) {
                if (containerSciFilterPattern.matcher(iter.next()).find()) {
                    iter.remove();
                }
            }
        }

        // Add the application services after the container services to ensure
        // that the container services are loaded first
        containerServicesFound.addAll(applicationServicesFound);

        // load the discovered services
        if (containerServicesFound.isEmpty()) {
            return Collections.emptyList();
        }
        return loadServices(serviceType, containerServicesFound);
    }
```



dubbo  SPI

自定义类加载器：隔离应用

webapp目录：放多个war

JDK  SPI

自定义类加载器：打破双亲委派

spring SPI

servlet3.0的核心接口：ServletContainerInitializers接口  调用 onStartup

为什么需要这个接口？

springmvc启动：

1、启动tomcat

2、完成IOC创建  初始化  扫描

tomcat.sh -->  webapp目录  war（spring）  servlet规范

ServletContainerInitializers接口实现类  任意jar

ServletContainerInitializers接口实现类放入standardContext全局变量initializers