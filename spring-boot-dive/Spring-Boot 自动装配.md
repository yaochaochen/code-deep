



# Spring-Boot 自动装配



## Spring模式注解装配

------

[模式注解]: https://github.com/spring-projects/spring-framework/wik

A **stereotype annotation** is an annotation that is used to declare the role that a component plays within the application. For example, the `@Repository` annotation in the Spring Framework is a marker for any class that fulfills the role or stereotype of a repository (also known as Data Access Object or DAO).

`@Component` is a generic stereotype for any Spring-managed component. Any component annotated with

`@Component` is a candidate for component scanning. Similarly, any component annotated with an annotation that is itself meta-annotated with `@Component` is also a candidate for component scanning. For example,

`@Service` is meta-annotated with `@Component .`

**模式注解**是一种用于声明在应用中扮演“组件”角色的注解。 如 Spring FrameWork 中的 `@Repository` 标注在任何类上 用于扮演仓储角色的模式

`@Component` 作为一种由 Spring 容器托管的通用模式组件，被任何 `@Component` 标准的组件均为组件扫描的候选对象。类似地， 凡是被 `@Component` 元标记（**meta-annotated**) 的注解，如 `@Service` 当任何组件标注它，也被是作为组件扫描候选对象。

------

### 模式注解举例

| Spring FrameWork注解 | 场景              | 起始版本 |
| -------------------- | ----------------- | -------- |
| `@Repository`        | 数据仓储模式注解  | 2.0      |
| `@Component`         | 通用组件模式注解  | 2.5      |
| `@Service`           | 服务模式注解      | 2.5      |
| `@Controller`        | Web控制器模式注解 | 2.5      |
| `@Configuration`     | 配置类模式注解    | 3.0      |



------

### 装配方式

#### <context:component-scan>方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring- context.xsd">
<!-- 激活注解驱动特性 --> <context:annotation-config />
<!-- 找寻被 @Component 或者其派生 Annotation 标记的类(Class)，将它们注册为 Spring Bean --> 		<context:component-scan base-package="com.imooc.dive.in.spring.boot" />
</beans>       
  	
```

------

```

 
```

#### @Component "派生性"

```java
 
/**
* 一级 {@link Repository @Repository}
*
* @author <a href="mailto:mercyblitz@gmail.com">Mercy</a> * @since 1.0.0
*/
@Target({ElementType.TYPE}) @Retention(RetentionPolicy.RUNTIME) @Documented
@Repository
public @interface FirstLevelRepository { String value() default "";
}
```

上代码关系

- `@Component`
  - `@Repository`
    - `@FirstLiveRepository`

#### `@Component` "层次性"

```java
@Target({ElementType.TYPE}) 
@Retention(RetentionPolicy.RUNTIME)
@Documented
@FirstLevelRepository
public @interface SecondLevelRepository { String value() default "";
}
```

- `@Component`
  - `@Repository`
    - `FistLeveRepository`
      - `SecondLeveRepository`

### Spring @Enable 模块装配

------

Spring FrameWork 3.1开始支持 "@Enable模块驱动"。所谓"模块"是指具备相同领域的功能组件集合，组合所形成一个独立的单元，比如Web MVC模块、Aspect代理模块、Caching 缓存模块、JMX Java管理扩展模块、Async异步处理模块等。

#### `@Enable` 注解举例

| 框架实现         | @Enable注解模块                  | 激活模块           |
| ---------------- | -------------------------------- | ------------------ |
| Spring Framework | `@EnableWebMvc`                  | Web MVC 模块       |
|                  | `@EnableTransactionManagent`     | 事务管理模块       |
|                  | `@EnableCaching`                 | Caching模块        |
|                  | `@EnableMBeanExport`             | JMX模块            |
|                  | `@EnableAsync`                   | 异步处理模块       |
|                  | `@EnableAspectJAutoproxy`        | ASpectJ代理模块    |
| Spring Boot      |                                  |                    |
|                  | `@EnableAutoConfiguration`       | 自动装配模块       |
|                  | `@EnableManagementContext`       | Actuator管理模块   |
|                  | `@EnableConfigurationProperties` | 配置属性绑定模块   |
|                  | `@EnableOAuth2Sso`               | OAuth2单点登录模块 |
|                  |                                  |                    |
| Spring Cloud     |                                  |                    |
|                  | `@EnableEurekaServer`            | Eureka服务器模块   |
|                  | `@EnableConfigServer`            | 配置服务器模块     |
|                  | `@EnableFeignClients`            | Feign客户端模块    |
|                  | `@EnableZuulProxy`               | 服务网关 Zuul模块  |
|                  | `@EnableCircuitBreaker`          | 服务熔断模块       |

#### 实现方式

#### 注解驱动方式

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class) 
public @interface EnableWebMvc {	
}
```

------

```java
@Configuration
public class DelegatingWebMvcConfiguration extends
WebMvcConfigurationSupport {
...
}
```

#### 接口编程方式

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({CachingConfigurationSelector.class})
public @interface EnableCaching {
  	...
}
```

```java
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {
    private static final String PROXY_JCACHE_CONFIGURATION_CLASS = "org.springframework.cache.jcache.config.ProxyJCacheConfiguration";
    private static final String CACHE_ASPECT_CONFIGURATION_CLASS_NAME = "org.springframework.cache.aspectj.AspectJCachingConfiguration";
    private static final String JCACHE_ASPECT_CONFIGURATION_CLASS_NAME = "org.springframework.cache.aspectj.AspectJJCacheConfiguration";
    private static final boolean jsr107Present = ClassUtils.isPresent("javax.cache.Cache", CachingConfigurationSelector.class.getClassLoader());
    private static final boolean jcacheImplPresent = ClassUtils.isPresent("org.springframework.cache.jcache.config.ProxyJCacheConfiguration", CachingConfigurationSelector.class.getClassLoader());

    public CachingConfigurationSelector() {
    }

    public String[] selectImports(AdviceMode adviceMode) {
        switch(adviceMode) {
        case PROXY:
            return this.getProxyImports();
        case ASPECTJ:
            return this.getAspectJImports();
        default:
            return null;
        }
    }

    private String[] getProxyImports() {
        List<String> result = new ArrayList();
        result.add(AutoProxyRegistrar.class.getName());
        result.add(ProxyCachingConfiguration.class.getName());
        if (jsr107Present && jcacheImplPresent) {
            result.add("org.springframework.cache.jcache.config.ProxyJCacheConfiguration");
        }

        return (String[])result.toArray(new String[result.size()]);
    }

    private String[] getAspectJImports() {
        List<String> result = new ArrayList();
        result.add("org.springframework.cache.aspectj.AspectJCachingConfiguration");
        if (jsr107Present && jcacheImplPresent) {
            result.add("org.springframework.cache.aspectj.AspectJJCacheConfiguration");
        }

        return (String[])result.toArray(new String[result.size()]);
    }
}
```

#### 自定义 `@Enable` 模块

##### ~~**基于驱动注解实现-`@EanbleHelloWorld`**~~ 

##### 基于接口驱动实现-`@EnableServer`

`HelloWorldImportSeletor`	-> `HelloWorldConfiguration` -> HelloWorld

### Spring 条件装配

从SpringFramework3.1开始，允许在Bean装配时增加前置条件判断

#### 条件注解举例

| Spring 注解  | 场景说明       | 起始版本 |
| ------------ | -------------- | -------- |
| `@Profile`   | 配置化条件装配 | 3.1      |
| `@Condition` | 编程条件装配   | 4.0      |

  

#### 实现方式

##### 配置方式-`@Profile`





##### 编程方式- `@Conditional`

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional({OnClassCondition.class})
public @interface ConditionalOnClass {
    Class<?>[] value() default {};

    String[] name() default {};
}
```

#### 自定义条件装配-`@Profile`

计算服务，多整数求和 sum

`@profile(""java7")` for

`@Profile("java8")` Lambda

##### 基于编程方式实现- `@ConditionalOnSystemProperty`

## Spring Boot自动装配

------

在Spring Boot场景下，基于约定大于配置的原则，实现Spring组件自动装配的目的

### 底层装配技术

- Spring模式注解装配
- Spring `@Enable`模块装配
- Spring条件装配
- Spring 工厂加载机制
  - 实现类: `SpringFactoriesLoader`
  - 配置资源: `META-INF/spring.factories`

### 实现方法

1. 激活自动装配-`@EnableAutoConfiguration`
2. 实现自动装配-`XXXAutoConfiguration`
3. 配置自动装配-`META-INF/spring.factories`

### 自定义自动装配

`HelloWoldAutoConfiguration`

- 条件判断 user.name="Yao"

- 模式注解 `@Configuration`

- `@Enable模块` `@EnableHelloWorld`->`HelloWorldImportSelector`->`HelloWorldConfiguration`->`helloWorld`

  