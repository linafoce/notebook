# 「12-06」Spring webflux系列之应用实战详解

# 一、Servlet线程模型比较



## 1、同步阻塞Servlet



- ```java
  /**
  
  
  
  - @Description:
  - @Author: 伯乐
  - @Date: 2020/11/17 19:51
    */
    [@WebServlet(name ](https://www.yuque.com/WebServlet(name) = "servlet2", urlPatterns = "/sync") 
    public class SyncServlet extends HttpServlet {
    protected void doGet(final HttpServletRequest request, final HttpServletResponse response) throws IOException {
    long start = System.currentTimeMillis();
    //处理业务
    doBusiness();
    //返回响应
    response.getWriter().write("sync Servlet");
    System.out.println("sync elapse time:" + (System.currentTimeMillis() - start));
    }
    /**
  
  - - 业务代码处理
      */
      private void doBusiness() {
      try {
      //模拟业务处理耗时10S
      TimeUnit.SECONDS.sleep(10);
      } catch (InterruptedException e) {
      e.printStackTrace();
      }
      }
      }
  ```

- 

- 测试结果：



业务线程阻塞10S



## 2、异步阻塞



开启线程异步处理



/**



- @Description:
- @Author: 伯乐
- @Date: 2020/11/17 20:41
  */
  [@WebServlet(name ](https://www.yuque.com/WebServlet(name) = "asyncServlet", value = "/async", asyncSupported = true) 
  public class AsyncServlet extends HttpServlet {
  [@Override ](https://www.yuque.com/Override) 
  protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException {
  long start = System.currentTimeMillis();
  //开启异步上下文
  AsyncContext asyncContext = req.startAsync();
  /* asyncContext.start(()->{
  try {
  asyncContext.getResponse().getWriter().write("hello async servlet");
  } catch (IOException e) {
  e.printStackTrace();
  }
  asyncContext.complete();
  });*/

```java
//开启一个线程处理业务
 new Thread(() -> {
     try {
         //业务处理
         doBusiness();
         //返回响应
         asyncContext.getResponse().getWriter().write(" async servlet");
     } catch (IOException e) {
         e.printStackTrace();
     }
     //异步结束
     asyncContext.complete();
 }).start();
```



/*

CompletableFuture.runAsync(()->{

try {

doBusiness();

asyncContext.getResponse().getWriter().write("Async servlet");

} catch (IOException e) {

e.printStackTrace();

}

});*/



```java
    System.out.println("async elapse time:" + (System.currentTimeMillis() - start));
}

/**
 * 业务处理
 */
private void doBusiness() {
    try {
        //模拟业务耗时10S
        TimeUnit.SECONDS.sleep(10);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```



}



## 3、异步非阻塞



/**



- @Description:
- @Author: 伯乐
- @Date: 2020/11/17 22:03
  */



@WebServlet(name="nioServlet",value = "/nio" ,asyncSupported = true)

public class NioServlet extends HttpServlet {



```
private static ExecutorService executorService=new ThreadPoolExecutor(10,10,5000, TimeUnit.MILLISECONDS,new ArrayBlockingQueue<>(100));
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    long start = System.currentTimeMillis();
    AsyncContext asyncContext = req.startAsync();
    ServletInputStream inputStream = req.getInputStream();
    inputStream.setReadListener(
            new ReadListener() {
                @Override
                public void onDataAvailable() throws IOException {
                    System.out.println("onDataAvailable");
                }

                @Override
                public void onAllDataRead() throws IOException {
                    System.out.println("onAllDataRead");
                    executorService.execute(()->{
                        try {
                            doBusiness();
                            asyncContext.getResponse().getWriter().write("Async read servlet");
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        asyncContext.complete();
                    });

                }

                @Override
                public void onError(Throwable t) {
                    System.out.println("onError");
                }
            }
    );

    System.out.println("nio elapse time:" + (System.currentTimeMillis() - start));
}

private void doBusiness() {
    try {
        TimeUnit.SECONDS.sleep(10);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```



}



# 二、Webflux快速开始





![image](https://cdn.nlark.com/yuque/0/2020/svg/1300531/1607328696502-2c61e51d-b6ea-4f73-b5c2-d0bf94f758f9.svg)





## 1、基于注解



### 添加依赖



<**dependency**>

  <**groupId**>org.springframework.boot</**groupId**>

  <**artifactId**>spring-boot-starter-webflux</**artifactId**>

</**dependency**>



### 基于注解代码



@SpringBootApplication

[@RestController ](https://www.yuque.com/RestController)

@RequestMapping("/annt")

public class AnnoControllerApplication {



```java
public static void main(String[] args) {
    SpringApplication.run(AnnoControllerApplication.class, args);
}

@GetMapping("/get")
public Mono<String> get() {
    return Mono.just("webflux annotation ");
}
```



}



## 2、函数编程



### 添加依赖



<**dependency**>

  <**groupId**>org.springframework.boot</**groupId**>

  <**artifactId**>spring-boot-starter-webflux</**artifactId**>

</**dependency**>





### 创建handler



[@Component ](https://www.yuque.com/Component)

public class WebfluxHandler {



```
public Mono<ServerResponse> hello(ServerRequest serverRequest) {
    return ServerResponse.ok()
            .contentType(MediaType.TEXT_PLAIN)
            .body(BodyInserters.fromValue("hello webflux"));
}
```



}





### 创建RouteFuntion 





[@Configuration ](https://www.yuque.com/Configuration)

public class WebflxuRouting {



```
@Bean
public RouterFunction<ServerResponse> route(WebfluxHandler webfluxHandler){
    return RouterFunctions.route(RequestPredicates.GET("/hello")
            .and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), webfluxHandler::hello)
            ;
}
```



}