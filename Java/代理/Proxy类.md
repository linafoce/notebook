## JDK动态代理

利用[JDK AP](https://tool.oschina.net/uploads/apidocs/jdk-zh/)I动态的在内存中构建**代理类的实例对象**

动态代理类（以下简称为代理类）是一个实现在创建类时在运行时指定的接口列表的类，该类具有下面描述的行为: 

- 代理接口 是代理类实现的一个接口。
- 代理实例 是代理类的一个实例。
- 每个代理实例都有一个关联的调用处理程序 对象，它可以实现接口 InvocationHandler。通过其中一个代理接口的代理实例上的方法调用将被指派到实例的调用处理程序的 Invoke 方法，并传递代理实例、识别调用方法的 java.lang.reflect.Method 对象以及包含参数的 Object 类型的数组。调用处理程序以适当的方式处理编码的方法调用，并且它返回的结果将作为代理实例上方法调用的结果返回



要了解 Java 动态代理的机制，首先需要了解以下相关的类或接口：

- `java.lang.reflect.Proxy`: 这是 Java 动态代理机制的主类，它提供了一组静态方法来为一组接口动态地生成代理类及其对象

Proxy 的静态方法

```java
    // 方法 1: 该方法用于获取指定代理对象所关联的调用处理器
    static InvocationHandler getInvocationHandler(Object proxy) 

    // 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象
    static Class getProxyClass(ClassLoader loader, Class[] interfaces) 

    // 方法 3：该方法用于判断指定类对象是否是一个动态代理类
    static boolean isProxyClass(Class cl) 

    // 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例
    static Object newProxyInstance(ClassLoader loader, Class[] interfaces, 
        InvocationHandler h)
复制代码
```

- `java.lang.reflect.InvocationHandler`：这是调用处理器接口，它自定义了一个 `invoke` 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问




作者：KodeMamba
链接：https://juejin.im/post/6844904003835265032
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。