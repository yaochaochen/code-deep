# 理解SpringApplication

## SpringApplication基本使用

### SpringApplication运行

```java
SpringApplication.run(DiveSpringBootApplication.class, args)
```

### 自定义SpringApplication

#### 通过SpringApplication API 调整

```java
 SpringApplication springApplication = new SpringApplication(DiveInSpringBootApplication.class); springApplication.setBannerMode(Banner.Mode.CONSOLE); springApplication.setWebApplicationType(WebApplicationType.NONE); springApplication.setAdditionalProfiles("prod");
springApplication.setHeadless(true);
```

#### 通过 SpringApplicationBuilder API 调整

```java
 new SpringApplicationBuilder(DiveInSpringBootApplication.class) .bannerMode(Banner.Mode.CONSOLE) .web(WebApplicationType.NONE)
.profiles("prod")
.headless(true) .run(args);
```

## SpringApplication准备阶段

### 配置Spring Boot Bean 源

Java配置Class 或者 XML 上下文配置文件集合， 用于 Spring Boot `BeanDefinitionLoader` 读取，并且将配置源解析加载为Spring Bean定义

- 数量: 一个或者多个以上

#### Java 配置Class 

用于Spring 注解驱动中 Java 配置类， 大多数情况是Spring 模式注解所标注的类，如 `@Configuration`

#### XML上下文配置文件

用于Spring 传统配置驱动中的 XML 文件

### 推断Web应用类型

根据当前应用ClassPath 中是否存在相关实现类来推断 Web 应用的类型

- Web Reactive ： `WebApplicationType.REACTIVE`
- Web Servlet : `WebApplicationType.SERVLET`
- 非Web： `WebApplicationType.NONE`

参考方法: `org.springframework.boot.SpringApplication#deduceWebApplicationType`

```java
 
private WebApplicationType deduceWebApplicationType() {
if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) { return WebApplicationType.REACTIVE;
    }
    for (String className : WEB_ENVIRONMENT_CLASSES) {
if (!ClassUtils.isPresent(className, null)) { return WebApplicationType.NONE;
} }
return WebApplicationType.SERVLET; }
```

### 推断引导类 （Main Class）

根据Main 线程执行堆栈判断实际的引导类

参考方法: `org.springframework.boot.SpringApplication#deduceMainApplicationClass`

```java
private Class<?> deduceMainApplicationClass() {
   try {
      StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
      for (StackTraceElement stackTraceElement : stackTrace) {
         if ("main".equals(stackTraceElement.getMethodName())) {
            return Class.forName(stackTraceElement.getClassName());
         }
      }
   }
   catch (ClassNotFoundException ex) {
      // Swallow and continue
   }
   return null;
}
```

### 加载 应用上下文初始器 （ApplicationContextInitializer）

利用Spring 工厂加载机制，实例化 `ApplicationContextInitalizer` 实现类， 并排序对象集合

- 实现

```java
 
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
  // Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(
		SpringFactoriesLoader.loadFactoryNames(type, classLoader)); List<T> instances = 				createSpringFactoriesInstances(type, parameterTypes,
		classLoader, args, names); AnnotationAwareOrderComparator.sort(instances);
  	return instances;

```

- 技术
  - 实现类： `org.springframework.core.io.support.SpringFactoriesLoader`
  - 配置资源： `META-INF/spring.factories`
  - 排序: `AnnotationAwareOrderComparator#sort`



#### 加载应用事件监听器( ApplicationListener )

 利用 Spring 工厂加载机制，实例化 ApplicationListener 实现类，并排序对象集合

## SpringApplication 运行阶段

### 加载SpringApplication 运行监听器 (SpringApplicationRunListener)

利用 Spring 工厂加载机制，读取`SpringApplicationRunListener` 对像集合，并且封装到组合类 `SpringApplicationRunListener` 

#### 运行SpringAPPlication 运行监听器 （`SpringApplicationRunListener`）



Spring Boot 通过 `SpringApplicationRunListener` 的实现类 `EventPublishingRunListener` 利用 Spring Framework 事件 API 广播 Spring Boot 事件。

#### Spring Framework 事件/监听器编程模型

- Spring 应用事件
  - 普通应用事件: `ApplicationEvent`
  - 应用上下文事件: `ApplicationContextEvent`
- Spring 应用监听器
  - 接口编程模型: `ApplicationListener`
  - 注解编程模型: `@EventListener`
- Spring 应用事件广播器
  - 接口: `ApplicationEventMulticaster`
  - 实现类: `SimpleApplicationEventMulticaster`
    - 执行模式:同步/异步

### `EventPublishingRunListener` 监听方法与 Spring Boot 事件关系

| 监听方法                                           | Spring Boot 事件                      | Spring Boot 起始版本 |
| -------------------------------------------------- | ------------------------------------- | -------------------- |
| staring()                                          | `ApplicationStartingEvent`            | 1.5                  |
| `eveironmentPrepared(ConfigurableEnvironment)`     | `ApplicationEnvironmentPreparedEvent` | 1.0                  |
| `contextPrepared(ConfigurableApplicationContext)`  |                                       |                      |
| `contextLoaded(ConfigurableApplicationContext)`    | `ApplicationPreparedEvent`            | 1.0                  |
| `started(ConfigurableApplicationContext)`          | `ApplicationStartedEvent`             | 2.0                  |
| `running(ConfigurableApplicationContext)`          | `ApplicationReadyEvent`               | 2.0                  |
| `failed(ConfigurableApplicationContext,Throwable)` | `ApplicationFailedEvent`              | 1.0                  |

### 创建 Spring 应用上下文( `ConfigurableApplicationContext` )

**根据准备阶段的推断 Web 应用类型创建对应的 ConfigurableApplicationContext 实例:**

- Web Reactive: `AnnotationConfigReactiveWebServerApplicationContext` 

- Web Servlet: `AnnotationConfigServletWebServerApplicationContext` 

- 非 Web: `AnnotationConfigApplicationContext`

#### 创建 Environment

 **根据准备阶段的推断 Web 应用类型创建对应的 ConfigurableEnvironment 实例:**

- Web Reactive: `StandardEnvironment`
  Web Servlet: `StandardServletEnvironment` 

- 非 Web: `StandardEnvironment`

