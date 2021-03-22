![image-20201126204406224](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201126204406224.png)

### @SpringBootApplication

```java
@Target(ElementType.TYPE)//修饰自定义注解，指定该自定义注解的注解位置，类还是方法，或者属性
@Retention(RetentionPolicy.RUNTIME)//被它所注解的注解保留多久，注解的生命周期。可选的参数值在枚举类型 RetentionPolicy 中,一般如果需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解；如果要在编译时进行一些预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS注解；如果只是做一些检查性的操作，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解。

@Documented//将此注解包含在 javadoc 中 ，它代表着此注解会被javadoc工具提取成文档。在doc文档中的内容会因为此注解的信息内容不同而不同。相当与@see（后面可以跟类路径等参数实现链接跳转  Ctrl跳转）,@param（注释参数） 等。
@Inherited//修饰自定义注解，该自定义注解注解的类，被继承时，子类也会拥有该自定义注解
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

1、RetentionPolicy.SOURCE：注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃；2、RetentionPolicy.CLASS：注解被保留到class文件，但jvm加载class文件时候被遗弃，这是默认的生命周期；

3、RetentionPolicy.RUNTIME：注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在；



#### @SpringBootConfiguration

相当于Configuration，表明启动类也是一个配置类，可以配置@Bean

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(annotation = Configuration.class)
    boolean proxyBeanMethods() default true;//默认使用CGLIB代理该类
}
```

#### @EnableAutoConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};

}
```

####  @AutoConfigurationPackage

保存自动配置类以供之后的使用，比如给`JPA entity`扫描器用来扫描开发人员通过注解`@Entity`定义的`entity`类。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)//手工注册bean
public @interface AutoConfigurationPackage {
```

##### @Import

- 普通类，直接导入到IOC容器管理
- ImportBeanDefinitionRegistrar接口实现类，支持手动注册bean
- ImportSelector实现，将selectImports方法返回的数组第加载到IOC

AutoConfigurationImportSelector实现了ImportSelector接口

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    // 获取注解的属性值
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 从META-INF/spring.factories文件中获取EnableAutoConfiguration所对应的configurations
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    // 去重
    configurations = removeDuplicates(configurations);
    // 从注解的exclude/excludeName属性中获取排除项
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    // 对于不属于AutoConfiguration的exclude报错
    checkExcludedClasses(configurations, exclusions);
    // 从configurations去除exclusions
    configurations.removeAll(exclusions);
    // 由所有AutoConfigurationImportFilter类的实例再进行一次筛选
    configurations = filter(configurations, autoConfigurationMetadata);
    // 把AutoConfigurationImportEvent绑定在所有AutoConfigurationImportListener子类实例上
    fireAutoConfigurationImportEvents(configurations, exclusions);
    // 返回(configurations, exclusions)组
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

## 自定义starter

1、新建两个模块：

​	命名规范，springboot自带的spring-boot-starter-xxx，自定义的xxx-spring-boot-starter

- xxx-spring-boot-autoconfigure：自动配置核心代码
- xxx-spring-boot-starter：管理依赖
  如果不需要将自动配置代码和依赖项管理分离开来，则可以将它们组合到一个模块中。

2、使用@ConfigurationProperties注入需要的配置

3、@Configuration + @Bean注册需要的bean,使用@EnableConfigurationProperties开启配置注入

4、利用META-INF/spring.factories导入@Configuration配置类（完全无侵入，但缺乏灵活性）

4.1、启动类使用@Import注解导入@Configuration配置类

4.1.1、`EnableXXX` 自定义注解使用@Import注解导入@Configuration配置类，启动类添加该注解