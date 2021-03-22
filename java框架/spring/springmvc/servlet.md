# servlet

通过url访问资源有三个步骤：

- 接收请求
- 处理请求
- 响应请求

web服务器：将某个主机上的资源映射为一个URL供外界访问，完成接收和响应请求

servlet容器：存放着servlet对象（由程序员编程提供），处理请求

## Servlet

```
public interface Servlet {
    //tomcat反射创建servlet之后，调用init方法传入ServletConfig
    void init(ServletConfig var1) throws ServletException;
    ServletConfig getServletConfig();
    //tomcat解析http请求，封装成对象传入
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
    String getServletInfo();
    void destroy();
}
```

ServletConfig：封装了servlet的参数信息，从web.xml中获取，init-param

ServletRequest：http请求到了tomcat后，tomcat通过字符串解析，把各个请求头（header），请求地址（URL），请求参数（queryString）都封装进Request。

ServletResponse：Response在tomcat传给servlet时还是空的对象，servlet逻辑处理后，最终通过response.write()方法，将结果写入response内部的缓冲区,tomcat会在servlet处理结束后拿到response，获取里面的信息，组装成http响应给客户端

## GenericServlet

改良版的servlet，抽象类

```
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
    private static final long serialVersionUID = 1L;
    private transient ServletConfig config;

    public GenericServlet() {
    }

    //并不是销毁servlet的方法，而是销毁servlet前一定会调用的方法。默认空实现,可以借此关闭一些资源
    public void destroy() {
    }

    public String getInitParameter(String name) {
        return this.getServletConfig().getInitParameter(name);
    }

    public Enumeration<String> getInitParameterNames() {
        return this.getServletConfig().getInitParameterNames();
    }

    public ServletConfig getServletConfig() {
        return this.config;//初始化时已被赋值
    }

    public ServletContext getServletContext() {
        //通过ServletConfig获取ServletContext
        return this.getServletConfig().getServletContext();
    }

    public String getServletInfo() {
        return "";
    }

    public void init(ServletConfig config) throws ServletException {
        this.config = config;//提升ServletConfig作用域，由局部变量变成全局变量
        this.init();//提供给子类覆盖
    }

    public void init() throws ServletException {
    }

    public void log(String message) {
        this.getServletContext().log(this.getServletName() + ": " + message);
    }

    public void log(String message, Throwable t) {
        this.getServletContext().log(this.getServletName() + ": " + message, t);
    }

    //空实现
    public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    public String getServletName() {
        return this.config.getServletName();
    }
}
```

HttpServlet

GenericServlet的升级版，针对http请求所定制，在GenericServlet的基础上增加了service方法的实现，完成请求方法的判断

抽象类，用来被子类继承，得到匹配http请求的处理，子类必须重写以下方法中的一个

doGet，doPost，doPut，doDelete 未重写会报错（400,405）

service方法不可以重写

模板模式实现

```
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    HttpServletRequest request;
    HttpServletResponse response;
    try {
        request = (HttpServletRequest)req;//强转成http类型，功能更强大
        response = (HttpServletResponse)res;
    } catch (ClassCastException var6) {
        throw new ServletException(lStrings.getString("http.non_http"));
    }

    this.service(request, response);//每次都调
}

protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String method = req.getMethod();//获取请求方式
    long lastModified;
    if (method.equals("GET")) {//判断逻辑，调用不同的处理方法
        lastModified = this.getLastModified(req);
        if (lastModified == -1L) {
            //本来业务逻辑应该直接写在这里，但是父类无法知道子类具体的业务逻辑，所以抽成方法让子类重写，父类的默认实现输出405，没有意义
            this.doGet(req, resp);
        } else {
            long ifModifiedSince;
            try {
                ifModifiedSince = req.getDateHeader("If-Modified-Since");
            } catch (IllegalArgumentException var9) {
                ifModifiedSince = -1L;
            }

            if (ifModifiedSince < lastModified / 1000L * 1000L) {
                this.maybeSetLastModified(resp, lastModified);
                this.doGet(req, resp);
            } else {
                resp.setStatus(304);
            }
        }
    } else if (method.equals("HEAD")) {
        lastModified = this.getLastModified(req);
        this.maybeSetLastModified(resp, lastModified);
        this.doHead(req, resp);
    } else if (method.equals("POST")) {
        this.doPost(req, resp);
    } else if (method.equals("PUT")) {
        this.doPut(req, resp);
    } else if (method.equals("DELETE")) {
        this.doDelete(req, resp);
    } else if (method.equals("OPTIONS")) {
        this.doOptions(req, resp);
    } else if (method.equals("TRACE")) {
        this.doTrace(req, resp);
    } else {
        String errMsg = lStrings.getString("http.method_not_implemented");
        Object[] errArgs = new Object[]{method};
        errMsg = MessageFormat.format(errMsg, errArgs);
        resp.sendError(501, errMsg);
    }

}
```

一个类被声明为抽象的，一般有两个原因：

- 有抽象方法需要被实现
- 没有抽象方法，但是不希望被实例化

## ServletContext

servlet上下文，代表web.xml文件，其实就是一个map，服务器会为每个应用创建一个servletContext对象：

- 创建是在服务器启动时完成
- 销毁是在服务器关闭时完成

javaWeb中的四个域对象：都可以看做是map，都有getAttribute()/setAttribute()方法。

- ServletContext域（Servlet间共享数据）
- Session域（一次会话间共享数据，也可以理解为多次请求间共享数据）
- Request域（同一次请求共享数据）
- Page域（JSP页面内共享数据）



servletConfig对象持有ServletContext的引用，Session域和Request域也可以得到ServletContext

五种方法获取：

\* ServletConfig#getServletContext();

\* GenericServlet#getServletContext();

\* HttpSession#getServletContext();

\* HttpServletRequest#getServletContext();

\* ServletContextEvent#getServletContext();//创建ioc容器时的监听



## Filter

不仅仅是拦截Request

拦截方式有四种：

![Filter.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/1618119/1605073230995-9a3e3819-c190-412b-b338-1f618ea4cce5.jpeg)

Redirect和REQUEST/FORWARD/INCLUDE/ERROR最大区别在于：

- 重定向会导致浏览器发送**2**次请求，FORWARD们是服务器内部的**1**次请求

因为FORWARD/INCLUDE等请求的分发是服务器内部的流程，不涉及浏览器，REQUEST/FORWARD/INCLUDE/ERROR和Request有关，Redirect通过Response发起

通过配置，Filter可以过滤服务器内部转发的请求



## servlet映射器

每一个url要交给哪个servlet处理，由映射器决定

![servlet映射器.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/1618119/1605073283063-de9f5a8e-319d-437b-9c8f-2d52be975439.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



映射器在tomcat中就是Mapper类：

internalMapWrapper方法定义了七种映射规则

```
private final void internalMapWrapper(ContextVersion contextVersion,
                                          CharChunk path,
                                          MappingData mappingData) throws IOException {

    int pathOffset = path.getOffset();
    int pathEnd = path.getEnd();
    boolean noServletPath = false;

    int length = contextVersion.path.length();
    if (length == (pathEnd - pathOffset)) {
        noServletPath = true;
    }
    int servletPath = pathOffset + length;
    path.setOffset(servletPath);

    // Rule 1 -- 精确匹配
    MappedWrapper[] exactWrappers = contextVersion.exactWrappers;
    internalMapExactWrapper(exactWrappers, path, mappingData);

    // Rule 2 -- 前缀匹配
    boolean checkJspWelcomeFiles = false;
    MappedWrapper[] wildcardWrappers = contextVersion.wildcardWrappers;
    if (mappingData.wrapper == null) {
        internalMapWildcardWrapper(wildcardWrappers, contextVersion.nesting,
                                   path, mappingData);
        if (mappingData.wrapper != null && mappingData.jspWildCard) {
            char[] buf = path.getBuffer();
            if (buf[pathEnd - 1] == '/') {
                mappingData.wrapper = null;
                checkJspWelcomeFiles = true;
            } else {
                // See Bugzilla 27704
                mappingData.wrapperPath.setChars(buf, path.getStart(),
                                                 path.getLength());
                mappingData.pathInfo.recycle();
            }
        }
    }

    if(mappingData.wrapper == null && noServletPath &&
       contextVersion.object.getMapperContextRootRedirectEnabled()) {
        // The path is empty, redirect to "/"
        path.append('/');
        pathEnd = path.getEnd();
        mappingData.redirectPath.setChars
            (path.getBuffer(), pathOffset, pathEnd - pathOffset);
        path.setEnd(pathEnd - 1);
        return;
    }

    // Rule 3 -- 扩展名匹配
    MappedWrapper[] extensionWrappers = contextVersion.extensionWrappers;
    if (mappingData.wrapper == null && !checkJspWelcomeFiles) {
        internalMapExtensionWrapper(extensionWrappers, path, mappingData,
                                    true);
    }

    ...
```

上面都不匹配，则交给DefaultServlet，就是简单地用IO流读取静态资源并响应给浏览器。如果资源找不到，报404错误

对于静态资源，Tomcat最后会交由一个叫做DefaultServlet的类来处理对于Servlet ，Tomcat最后会交由一个叫做 InvokerServlet的类来处理对于JSP，Tomcat最后会交由一个叫做JspServlet的类来处理



![tomcat.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/1618119/1605073328747-6a6269f0-2e05-473f-a9f2-4c28e2ddc206.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_14%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



也就是说，servlet,/*这种配置，相当于把DefaultServlet、JspServlet以及我们自己写的其他Servlet都“短路”了，它们都失效了。

这会导致两个问题：

- JSP无法被编译成Servlet输出HTML片段（JspServlet短路）
- HTML/CSS/JS/PNG等资源无法获取（DefaultServlet短路）

DispatcherServlet配置/，会和DefaultServlet产生路径冲突，从而覆盖DefaultServlet。此时，所有对静态资源的请求，映射器都会分发给我们自己写的DispatcherServlet处理。遗憾的是，它只写了业务代码，并不能IO读取并返回静态资源。JspServlet的映射路径没有被覆盖，所以动态资源照常响应。

DispatcherServlet配置/*，虽然JspServlet和DefaultServlet拦截路径还是.jsp和/，没有被覆盖，但无奈的是在到达它们之前，请求已经被DispatcherServlet抢去，所以最终不仅无法处理JSP，也无法处理静态资源。

tomcat中conf/web.xml

相当于每个应用默认都配置了JSPServlet和DefaultServlet处理JSP和静态资源。



```
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
        <param-name>debug</param-name>
        <param-value>0</param-value>
    </init-param>
    <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>


<servlet>
    <servlet-name>jsp</servlet-name>
    <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
    <init-param>
        <param-name>fork</param-name>
        <param-value>false</param-value>
    </init-param>
    <init-param>
        <param-name>xpoweredBy</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>3</load-on-startup>
</servlet>

<!-- The mappings for the JSP servlet -->
<servlet-mapping>
    <servlet-name>jsp</servlet-name>
    <url-pattern>*.jsp</url-pattern>
    <url-pattern>*.jspx</url-pattern>
</servlet-mapping>
```

![servlet.png](https://cdn.nlark.com/yuque/0/2020/png/1618119/1605504691696-437576a3-b72d-4ecd-9c5a-20a69beb5481.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_14%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)



生命周期：

创建

init(ServletConfig config)

service   处理请求

destroy



servletContext

模板模式



GenericServlet：将ServletConfig从局部变量变成全局变量

HttpServlet：针对http请求定制的servlet

http：get  post

**增删改查  必须重写其中之一**

<code>doGet</code>, if the servlet supports HTTP GET requests

<code>doPost</code>, for HTTP POST requests

<code>doPut</code>, for HTTP PUT requests

<code>doDelete</code>, for HTTP DELETE requests

```java
/**
servlet会区分请求
*/
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

DispatcherServlet

DefaultServlet: 兜底，默认的

jspServlet



Jsp： 动态资源

html：静态资源

# 课堂笔记

springmvc4  +  springboot4

dispatcherServlet

servlet

图片：url  http：。。。。。。

http  https协议

web服务器：把资源映射成url

1、接收请求---统一的解析   ServletRequest

2、处理请求  servlet提供给程序员开发

3、响应请求---   ServletResponse

tomcat：1、3  web服务器  + servlet容器

servlet + jsp

servlet规范  接口定义  javaweb

生命周期：

创建

init(ServletConfig config)

service   处理请求

destroy

servletContext

模板模式

GenericServlet：将ServletConfig从局部变量变成全局变量

HttpServlet：针对http请求定制的servlet

http：get  post

增删改查  必须重写其中之一

<code>doGet</code>, if the servlet supports HTTP GET requests

<code>doPost</code>, for HTTP POST requests

<code>doPut</code>, for HTTP PUT requests

<code>doDelete</code>, for HTTP DELETE requests

400 405

service  不能重写

DispatcherServlet

DefaultServlet（默认的，缺省）:兜底

jspServlet  jasper

处理器

jsp：动态资源

html：静态资源

视图解析器：前后端分离

所有请求：

/  和  /*（优先级高些）

/*  *.jsp

classpath:  class路径  （不找jar包）  

classpath*：

源码：

servlet初始化

service怎么被调用的

监听器模式   

addWebapp

1. Server  --->  Service  -->  Connector  Engine addChild--->  context(servlet容器)

   tomcat隔层容器实例化

   ```java
   public Host getHost() {
           Engine engine = getEngine();
           if (engine.findChildren().length > 0) {
               return (Host) engine.findChildren()[0];
           }
   
           Host host = new StandardHost();
           host.setName(hostname);
           getEngine().addChild(host);
           return host;
       }
   ```

2. 添加服务器

   ```java
   /**
        * @param host The host in which the context will be deployed
        * @param contextPath The context mapping to use, "" for root context.
        * @param docBase Base directory for the context, for static files.
        *  Must exist, relative to the server home
        * @return the deployed context
        * @see #addWebapp(String, String)
        */
       public Context addWebapp(Host host, String contextPath, String docBase) {
         	//定义监听器
           LifecycleListener listener = null;
           try {
             	 /**
        * The Java class name of the default context configuration class
        * for deployed web applications.
        用的是这config，也是上面接口的实现类
         "org.apache.catalina.startup.ContextConfig"
        */
               Class<?> clazz = Class.forName(getHost().getConfigClass());
               listener = (LifecycleListener) clazz.getConstructor().newInstance();
           } catch (ReflectiveOperationException e) {
               // Wrap in IAE since we can't easily change the method signature to
               // to throw the specific checked exceptions
               throw new IllegalArgumentException(e);
           }
   
           return addWebapp(host,  contextPath, docBase, listener);
       }
   ```

   ```java
   /**
        * @param host The host in which the context will be deployed
        * @param contextPath The context mapping to use, "" for root context.
        * @param docBase Base directory for the context, for static files.
        *  Must exist, relative to the server home
        * @param config Custom context configurator helper
        * @return the deployed context
        * @see #addWebapp(String, String)
        */
       public Context addWebapp(Host host, String contextPath, String docBase,
               LifecycleListener config) {
   
           silence(host, contextPath);
   				//创建容器结构
           Context ctx = createContext(host, contextPath);
           ctx.setPath(contextPath);
           ctx.setDocBase(docBase);
           ctx.addLifecycleListener(getDefaultWebXmlListener());
           ctx.setConfigFile(getWebappConfigFile(docBase, contextPath));
   
           ctx.addLifecycleListener(config);
   
           if (config instanceof ContextConfig) {
               // prevent it from looking ( if it finds one - it'll have dup error )
               ((ContextConfig) config).setDefaultWebXml(noDefaultWebXmlPath());
           }
   
           if (host == null) {
             	// 这里面会掉自己Tomcat类里面继承的addChild
               getHost().addChild(ctx);
           } else {
               host.addChild(ctx);
           }
   
           return ctx;
       }
   ```

   ```java
   	/**
   	这一步就是把context加入 container里面
   	*/
     public void addChild(Container child) {
           if (Globals.IS_SECURITY_ENABLED) {
               PrivilegedAction<Void> dp =
                   new PrivilegedAddChild(child);
               AccessController.doPrivileged(dp);
           } else {
               addChildInternal(child);
           }
       }
   
       private void addChildInternal(Container child) {
   
           if( log.isDebugEnabled() )
               log.debug("Add child " + child + " " + this);
           synchronized(children) {
               if (children.get(child.getName()) != null)
                   throw new IllegalArgumentException("addChild:  Child name '" +
                                                      child.getName() +
                                                      "' is not unique");
               child.setParent(this);  // May throw IAE
               children.put(child.getName(), child);
           }
   
           // Start child
           // Don't do this inside sync block - start can be a slow process and
           // locking the children object can cause problems elsewhere
           try {
               if ((getState().isAvailable() ||
                       LifecycleState.STARTING_PREP.equals(getState())) &&
                       startChildren) {
                   child.start();
               }
           } catch (LifecycleException e) {
               log.error("ContainerBase.addChild: start: ", e);
               throw new IllegalStateException("ContainerBase.addChild: start: " + e);
           } finally {
               fireContainerEvent(ADD_CHILD_EVENT, child);
           }
       }
   ```

   

3. Start()启动

   

   ```java
   /**
    * Start the server.
    * 
    * @throws LifecycleException Start error
    */
   public void start() throws LifecycleException {
       //会判断有没有实例化，没有实例化给默认的
       getServer();
       //实例化connecter
       getConnector();
       server.start();
   }
   ```

   ```java
   /**
        * {@inheritDoc}
        子类没有重写，一般都是调用的父类里面的start方法
        */
       @Override
       public final synchronized void start() throws LifecycleException {
   
           if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
                   LifecycleState.STARTED.equals(state)) {
   
               if (log.isDebugEnabled()) {
                   Exception e = new LifecycleException();
                   log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
               } else if (log.isInfoEnabled()) {
                   log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
               }
   
               return;
           }
   
           if (state.equals(LifecycleState.NEW)) {
               init();
           } else if (state.equals(LifecycleState.FAILED)) {
               stop();
           } else if (!state.equals(LifecycleState.INITIALIZED) &&
                   !state.equals(LifecycleState.STOPPED)) {
               invalidTransition(Lifecycle.BEFORE_START_EVENT);
           }
   
           try {
               setStateInternal(LifecycleState.STARTING_PREP, null, false);
               // 这里面一层一层往下走，各个容器全部启动
               // 实现类在standardserver里面
               startInternal();
               if (state.equals(LifecycleState.FAILED)) {
                   // This is a 'controlled' failure. The component put itself into the
                   // FAILED state so call stop() to complete the clean-up.
                   stop();
               } else if (!state.equals(LifecycleState.STARTING)) {
                   // Shouldn't be necessary but acts as a check that sub-classes are
                   // doing what they are supposed to.
                   invalidTransition(Lifecycle.AFTER_START_EVENT);
               } else {
                   setStateInternal(LifecycleState.STARTED, null, false);
               }
           } catch (Throwable t) {
               // This is an 'uncontrolled' failure so put the component into the
               // FAILED state and throw an exception.
               ExceptionUtils.handleThrowable(t);
               setStateInternal(LifecycleState.FAILED, null, false);
               throw new LifecycleException(sm.getString("lifecycleBase.startFail", toString()), t);
           }
       }
   ```

   ```java
   /**
        * {@inheritDoc}
        */
       @Override
       public final synchronized void start() throws LifecycleException {
   
           if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
                   LifecycleState.STARTED.equals(state)) {
   
               if (log.isDebugEnabled()) {
                   Exception e = new LifecycleException();
                   log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
               } else if (log.isInfoEnabled()) {
                   log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
               }
   
               return;
           }
   
           if (state.equals(LifecycleState.NEW)) {
               init();
           } else if (state.equals(LifecycleState.FAILED)) {
               stop();
           } else if (!state.equals(LifecycleState.INITIALIZED) &&
                   !state.equals(LifecycleState.STOPPED)) {
               invalidTransition(Lifecycle.BEFORE_START_EVENT);
           }
   
           try {
               setStateInternal(LifecycleState.STARTING_PREP, null, false);
               startInternal();
               if (state.equals(LifecycleState.FAILED)) {
                   // This is a 'controlled' failure. The component put itself into the
                   // FAILED state so call stop() to complete the clean-up.
                   stop();
               } else if (!state.equals(LifecycleState.STARTING)) {
                   // Shouldn't be necessary but acts as a check that sub-classes are
                   // doing what they are supposed to.
                   invalidTransition(Lifecycle.AFTER_START_EVENT);
               } else {
                   setStateInternal(LifecycleState.STARTED, null, false);
               }
           } catch (Throwable t) {
               // This is an 'uncontrolled' failure so put the component into the
               // FAILED state and throw an exception.
               ExceptionUtils.handleThrowable(t);
               setStateInternal(LifecycleState.FAILED, null, false);
               throw new LifecycleException(sm.getString("lifecycleBase.startFail", toString()), t);
           }
       }
   ```

   **类StandServer**

   ```java
   /**
        * Start nested components ({@link Service}s) and implement the requirements
        * of {@link org.apache.catalina.util.LifecycleBase#startInternal()}.
        *
        * @exception LifecycleException if this component detects a fatal error
        *  that prevents this component from being used
        */
       @Override
       protected void startInternal() throws LifecycleException {
   
           fireLifecycleEvent(CONFIGURE_START_EVENT, null);
           setState(LifecycleState.STARTING);
   
           globalNamingResources.start();
   
           // Start our defined Services
          //有可能多个service
           synchronized (servicesLock) {
               for (int i = 0; i < services.length; i++) {
                   //这个地方很关键，第一次进去的时候是走的server启动
                   //这次是service里面启动
                   //所以startInternal的实现类就不一样了,是StandardService里面的实现类
                   services[i].start();
               }
           }
       }
   ```

   ## 类StandardService

   ```java
   /**
        * Start nested components ({@link Executor}s, {@link Connector}s and
        * {@link Container}s) and implement the requirements of
        * {@link org.apache.catalina.util.LifecycleBase#startInternal()}.
        *
        * @exception LifecycleException if this component detects a fatal error
        *  that prevents this component from being used
        */
       @Override
       protected void startInternal() throws LifecycleException {
   
           if(log.isInfoEnabled())
               log.info(sm.getString("standardService.start.name", this.name));
           setState(LifecycleState.STARTING);
   
           // Start our defined Container first
           if (engine != null) {
               synchronized (engine) {
                   //再次调用start方法
                   engine.start();
               }
           }
   
           synchronized (executors) {
               for (Executor executor: executors) {
                   executor.start();
               }
           }
   
           mapperListener.start();
   
           // Start our defined Connectors second
           synchronized (connectorsLock) {
               for (Connector connector: connectors) {
                   try {
                       // If it has already failed, don't try and start it
                       if (connector.getState() != LifecycleState.FAILED) {
                         //再次调用start方法
                           connector.start();
                       }
                   } catch (Exception e) {
                       log.error(sm.getString(
                               "standardService.connector.startFailed",
                               connector), e);
                   }
               }
           }
       }
   ```

   ## 类Connector

   ```java
   /**
        * Begin processing requests via this Connector.
        *
        * @exception LifecycleException if a fatal startup error occurs
        connector的start方法
        */
       @Override
       protected void startInternal() throws LifecycleException {
   
           // Validate settings before starting
           if (getPort() < 0) {
               throw new LifecycleException(sm.getString(
                       "coyoteConnector.invalidPort", Integer.valueOf(getPort())));
           }
   
           setState(LifecycleState.STARTING);
   
           try {
               //这里面的start，就会调用nio，进行socket监听
               protocolHandler.start();
           } catch (Exception e) {
               throw new LifecycleException(
                       sm.getString("coyoteConnector.protocolHandlerStartFailed"), e);
           }
       }
   ```

   ## 类NioEndpoint

   ```java
   Override
       public void startInternal() throws Exception {
   
           if (!running) {
               running = true;
               paused = false;
   
               processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                       socketProperties.getProcessorCache());
               eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                               socketProperties.getEventCache());
               nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                       socketProperties.getBufferPool());
   
               // Create worker collection
               if ( getExecutor() == null ) {
                   createExecutor();
               }
   
               initializeConnectionLatch();
   
               // Start poller threads
               //这个Poller继承了runnable接口，调用start()，内部就会跑run方法，开启nio监听
               pollers = new Poller[getPollerThreadCount()];
               for (int i=0; i<pollers.length; i++) {
                   pollers[i] = new Poller();
                   Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
                   pollerThread.setPriority(threadPriority);
                   pollerThread.setDaemon(true);
                   pollerThread.start();
               }
   
               startAcceptorThreads();
           }
       }
   ```

4.  StandartEngine的start方法

   ```java
   /**
        * Start this component and implement the requirements
        * of {@link org.apache.catalina.util.LifecycleBase#startInternal()}.
        *
        * @exception LifecycleException if this component detects a fatal error
        *  that prevents this component from being used
        servletContext 是在存在容器内部的，所以这里面有一个startchild来启动startContext
        */
       @Override
       protected synchronized void startInternal() throws LifecycleException {
   
           // Start our subordinate components, if any
           logger = null;
           getLogger();
           Cluster cluster = getClusterInternal();
           if (cluster instanceof Lifecycle) {
               ((Lifecycle) cluster).start();
           }
           Realm realm = getRealmInternal();
           if (realm instanceof Lifecycle) {
               ((Lifecycle) realm).start();
           }
   
           // Start our child containers, if any
           Container children[] = findChildren();
           List<Future<Void>> results = new ArrayList<>();
           for (int i = 0; i < children.length; i++) {
               //这里会start,ServletContext
               results.add(startStopExecutor.submit(new StartChild(children[i])));
           }
   
           boolean fail = false;
           for (Future<Void> result : results) {
               try {
                   result.get();
               } catch (Exception e) {
                   log.error(sm.getString("containerBase.threadedStartFailed"), e);
                   fail = true;
               }
   
           }
           if (fail) {
               throw new LifecycleException(
                       sm.getString("containerBase.threadedStartFailed"));
           }
   
           // Start the Valves in our pipeline (including the basic), if any
           if (pipeline instanceof Lifecycle)
               ((Lifecycle) pipeline).start();
   
   
           setState(LifecycleState.STARTING);
   
           // Start our thread
           threadStart();
   
       }
   ```

   ## 类StandartContext

   ```java
   /**
        * Start this component and implement the requirements
        * of {@link org.apache.catalina.util.LifecycleBase#startInternal()}.
        *
        * @exception LifecycleException if this component detects a fatal error
        *  that prevents this component from being used
        */
       @Override
       protected synchronized void startInternal() throws LifecycleException {
   
           if(log.isDebugEnabled())
               log.debug("Starting " + getBaseName());
   
           // Send j2ee.state.starting notification
           if (this.getObjectName() != null) {
               Notification notification = new Notification("j2ee.state.starting",
                       this.getObjectName(), sequenceNumber.getAndIncrement());
               broadcaster.sendNotification(notification);
           }
   
           setConfigured(false);
           boolean ok = true;
   
           // Currently this is effectively a NO-OP but needs to be called to
           // ensure the NamingResources follows the correct lifecycle
           if (namingResources != null) {
               namingResources.start();
           }
   
           // Post work directory
           postWorkDirectory();
   
           // Add missing components as necessary
           if (getResources() == null) {   // (1) Required by Loader
               if (log.isDebugEnabled())
                   log.debug("Configuring default Resources");
   
               try {
                   setResources(new StandardRoot(this));
               } catch (IllegalArgumentException e) {
                   log.error(sm.getString("standardContext.resourcesInit"), e);
                   ok = false;
               }
           }
           if (ok) {
               resourcesStart();
           }
   
           if (getLoader() == null) {
               WebappLoader webappLoader = new WebappLoader(getParentClassLoader());
               webappLoader.setDelegate(getDelegate());
               setLoader(webappLoader);
           }
   
           // An explicit cookie processor hasn't been specified; use the default
           if (cookieProcessor == null) {
               cookieProcessor = new Rfc6265CookieProcessor();
           }
   
           // Initialize character set mapper
           getCharsetMapper();
   
           // Validate required extensions
           boolean dependencyCheck = true;
           try {
               dependencyCheck = ExtensionValidator.validateApplication
                   (getResources(), this);
           } catch (IOException ioe) {
               log.error(sm.getString("standardContext.extensionValidationError"), ioe);
               dependencyCheck = false;
           }
   
           if (!dependencyCheck) {
               // do not make application available if dependency check fails
               ok = false;
           }
   
           // Reading the "catalina.useNaming" environment variable
           String useNamingProperty = System.getProperty("catalina.useNaming");
           if ((useNamingProperty != null)
               && (useNamingProperty.equals("false"))) {
               useNaming = false;
           }
   
           if (ok && isUseNaming()) {
               if (getNamingContextListener() == null) {
                   NamingContextListener ncl = new NamingContextListener();
                   ncl.setName(getNamingContextName());
                   ncl.setExceptionOnFailedWrite(getJndiExceptionOnFailedWrite());
                   addLifecycleListener(ncl);
                   setNamingContextListener(ncl);
               }
           }
   
           // Standard container startup
           if (log.isDebugEnabled())
               log.debug("Processing standard container startup");
   
   
           // Binding thread
           ClassLoader oldCCL = bindThread();
   
           try {
               if (ok) {
                   // Start our subordinate components, if any
                   Loader loader = getLoader();
                   if (loader instanceof Lifecycle) {
                       ((Lifecycle) loader).start();
                   }
   
                   // since the loader just started, the webapp classloader is now
                   // created.
                   setClassLoaderProperty("clearReferencesRmiTargets",
                           getClearReferencesRmiTargets());
                   setClassLoaderProperty("clearReferencesStopThreads",
                           getClearReferencesStopThreads());
                   setClassLoaderProperty("clearReferencesStopTimerThreads",
                           getClearReferencesStopTimerThreads());
                   setClassLoaderProperty("clearReferencesHttpClientKeepAliveThread",
                           getClearReferencesHttpClientKeepAliveThread());
                   setClassLoaderProperty("clearReferencesObjectStreamClassCaches",
                           getClearReferencesObjectStreamClassCaches());
   
                   // By calling unbindThread and bindThread in a row, we setup the
                   // current Thread CCL to be the webapp classloader
                   unbindThread(oldCCL);
                   oldCCL = bindThread();
   
                   // Initialize logger again. Other components might have used it
                   // too early, so it should be reset.
                   logger = null;
                   getLogger();
   
                   Realm realm = getRealmInternal();
                   if(null != realm) {
                       if (realm instanceof Lifecycle) {
                           ((Lifecycle) realm).start();
                       }
   
                       // Place the CredentialHandler into the ServletContext so
                       // applications can have access to it. Wrap it in a "safe"
                       // handler so application's can't modify it.
                       CredentialHandler safeHandler = new CredentialHandler() {
                           @Override
                           public boolean matches(String inputCredentials, String storedCredentials) {
                               return getRealmInternal().getCredentialHandler().matches(inputCredentials, storedCredentials);
                           }
   
                           @Override
                           public String mutate(String inputCredentials) {
                               return getRealmInternal().getCredentialHandler().mutate(inputCredentials);
                           }
                       };
                       context.setAttribute(Globals.CREDENTIAL_HANDLER, safeHandler);
                   }
   
                   // Notify our interested LifecycleListeners
                   fireLifecycleEvent(Lifecycle.CONFIGURE_START_EVENT, null);
   
                   // Start our child containers, if not already started
                   for (Container child : findChildren()) {
                       if (!child.getState().isAvailable()) {
                           child.start();
                       }
                   }
   
                   // Start the Valves in our pipeline (including the basic),
                   // if any
                   if (pipeline instanceof Lifecycle) {
                       ((Lifecycle) pipeline).start();
                   }
   
                   // Acquire clustered manager
                   Manager contextManager = null;
                   Manager manager = getManager();
                   if (manager == null) {
                       if (log.isDebugEnabled()) {
                           log.debug(sm.getString("standardContext.cluster.noManager",
                                   Boolean.valueOf((getCluster() != null)),
                                   Boolean.valueOf(distributable)));
                       }
                       if ( (getCluster() != null) && distributable) {
                           try {
                               contextManager = getCluster().createManager(getName());
                           } catch (Exception ex) {
                               log.error("standardContext.clusterFail", ex);
                               ok = false;
                           }
                       } else {
                           contextManager = new StandardManager();
                       }
                   }
   
                   // Configure default manager if none was specified
                   if (contextManager != null) {
                       if (log.isDebugEnabled()) {
                           log.debug(sm.getString("standardContext.manager",
                                   contextManager.getClass().getName()));
                       }
                       setManager(contextManager);
                   }
   
                   if (manager!=null && (getCluster() != null) && distributable) {
                       //let the cluster know that there is a context that is distributable
                       //and that it has its own manager
                       getCluster().registerManager(manager);
                   }
               }
   
               if (!getConfigured()) {
                   log.error(sm.getString("standardContext.configurationFail"));
                   ok = false;
               }
   
               // We put the resources into the servlet context
               if (ok)
                   getServletContext().setAttribute
                       (Globals.RESOURCES_ATTR, getResources());
   
               if (ok ) {
                   if (getInstanceManager() == null) {
                       javax.naming.Context context = null;
                       if (isUseNaming() && getNamingContextListener() != null) {
                           context = getNamingContextListener().getEnvContext();
                       }
                       Map<String, Map<String, String>> injectionMap = buildInjectionMap(
                               getIgnoreAnnotations() ? new NamingResourcesImpl(): getNamingResources());
                       setInstanceManager(new DefaultInstanceManager(context,
                               injectionMap, this, this.getClass().getClassLoader()));
                   }
                   getServletContext().setAttribute(
                           InstanceManager.class.getName(), getInstanceManager());
                   InstanceManagerBindings.bind(getLoader().getClassLoader(), getInstanceManager());
               }
   
               // Create context attributes that will be required
               if (ok) {
                   getServletContext().setAttribute(
                           JarScanner.class.getName(), getJarScanner());
               }
   
               // Set up the context init params
               mergeParameters();
   
               // Call ServletContainerInitializers
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
   
               // Configure and call application event listeners
               if (ok) {
                   if (!listenerStart()) {
                       log.error(sm.getString("standardContext.listenerFail"));
                       ok = false;
                   }
               }
   
               // Check constraints for uncovered HTTP methods
               // Needs to be after SCIs and listeners as they may programmatically
               // change constraints
               if (ok) {
                   checkConstraintsForUncoveredMethods(findConstraints());
               }
   
               try {
                   // Start manager
                   Manager manager = getManager();
                   if (manager instanceof Lifecycle) {
                       ((Lifecycle) manager).start();
                   }
               } catch(Exception e) {
                   log.error(sm.getString("standardContext.managerFail"), e);
                   ok = false;
               }
   
               // Configure and call application filters
               if (ok) {
                   if (!filterStart()) {
                       log.error(sm.getString("standardContext.filterFail"));
                       ok = false;
                   }
               }
   
               // Load and initialize all "load on startup" servlets
               if (ok) {
                   if (!loadOnStartup(findChildren())){
                       log.error(sm.getString("standardContext.servletFail"));
                       ok = false;
                   }
               }
   
               // Start ContainerBackgroundProcessor thread
               super.threadStart();
           } finally {
               // Unbinding thread
               unbindThread(oldCCL);
           }
   
           // Set available status depending upon startup success
           if (ok) {
               if (log.isDebugEnabled())
                   log.debug("Starting completed");
           } else {
               log.error(sm.getString("standardContext.startFailed", getName()));
           }
   
           startTime=System.currentTimeMillis();
   
           // Send j2ee.state.running notification
           if (ok && (this.getObjectName() != null)) {
               Notification notification =
                   new Notification("j2ee.state.running", this.getObjectName(),
                                    sequenceNumber.getAndIncrement());
               broadcaster.sendNotification(notification);
           }
   
           // The WebResources implementation caches references to JAR files. On
           // some platforms these references may lock the JAR files. Since web
           // application start is likely to have read from lots of JARs, trigger
           // a clean-up now.
           getResources().gc();
   
           // Reinitializing if something went wrong
           if (!ok) {
               setState(LifecycleState.FAILED);
           } else {
               setState(LifecycleState.STARTING);
           }
       }
   ```

   

5. 调用监听器方法

   ```java
   /**
        * Process events for an associated Context.
        *
        * @param event The lifecycle event that has occurred
        主要是解析xml文件
        通过监听器完成servlet包装成wrapper，并且方到context里面
        */
       @Override
       public void lifecycleEvent(LifecycleEvent event) {
   
           // Identify the context we are associated with
           try {
               context = (Context) event.getLifecycle();
           } catch (ClassCastException e) {
               log.error(sm.getString("contextConfig.cce", event.getLifecycle()), e);
               return;
           }
   
           // Process the event that has occurred
           if (event.getType().equals(Lifecycle.CONFIGURE_START_EVENT)) {
               configureStart();
           } else if (event.getType().equals(Lifecycle.BEFORE_START_EVENT)) {
               beforeStart();
           } else if (event.getType().equals(Lifecycle.AFTER_START_EVENT)) {
               // Restore docBase for management tools
               if (originalDocBase != null) {
                   context.setDocBase(originalDocBase);
               }
           } else if (event.getType().equals(Lifecycle.CONFIGURE_STOP_EVENT)) {
               configureStop();
           } else if (event.getType().equals(Lifecycle.AFTER_INIT_EVENT)) {
               init();
           } else if (event.getType().equals(Lifecycle.AFTER_DESTROY_EVENT)) {
               destroy();
           }
   
       }
   ```

   

6. 

