# Spring IoC 容器

## BeanFactory 和ApplicationContext 谁才是 Spring IoC容器？

**AppliactionContext 就是 BeanFactory在官方文档里可以找出答案**



> This chapter covers the Spring Framework implementation of the Inversion of Control (IoC) principle. IoC is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.
>
> The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Framework’s IoC container. The [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html) interface provides an advanced configuration mechanism capable of managing any type of object. [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html) is a sub-interface of `BeanFactory`. It adds:
>
> - Easier integration with Spring’s AOP features
> - Message resource handling (for use in internationalization)
> - Event publication
> - Application-layer specific contexts such as the `WebApplicationContext` for use in web applications.
>
> In short, the `BeanFactory` provides the configuration framework and basic functionality, and the `ApplicationContext` adds more enterprise-specific functionality. The `ApplicationContext` is a complete superset of the `BeanFactory` and is used exclusively in this chapter in descriptions of Spring’s IoC container. For more information on using the `BeanFactory` instead of the `ApplicationContext,` see [[beans-beanfactory\]](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-beanfactory).

​	

[https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#spring-core]: 	"Spring IoC"

BeanFactory 是个提供配置框架和基本功能，ApplicationContext 是 BeanFactory 的子接口

## 源码分析

ConfigurableApplicationContext  -> ApplicationContext <- BeanFactory

`ConfigurableApplicationContext`#`getBeanFactory`

在上下文应用里组合模式+继承关系 

BeanFactory 是底层的 IoC 容器 虽然有继承关系 但是是2个对象

## 使用 Spring IoC 容器

- BeanFactory 是 Spring 底层 IoC 容器

  ```java
  //创建 beanFactory 容器
  DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
  XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
  //classpath:/META-INF/dependency-injection-context.xml
  String classPath = "classpath:/META-INF/dependency-injection-context.xml";
  //加载资源
  int beanCount =  reader.loadBeanDefinitions(classPath);
  System.out.println(" Bean 的个数 " + beanCount);
  ```

- ApplicationContext 是具备应用特性 BeanFactory 超集

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.register(AnnotationsApplicationContextAsIoCContainerDemo.class);
    applicationContext.refresh();
    lookupCollectionByType(applicationContext);
}

@Bean
public User user() {
    User user = new User();
    user.setId(1L);
    user.setName("Spring-Dive");
    return user;
}
```

## IoC 生命周期概述

- 启动

  ```java
  applicationContext.register(AnnotationsApplicationContextAsIoCContainerDemo.class);
  ```

- 运行

  ```java
  applicationContext.refresh();
  ```

- 停止

  ```java
  applicationContext.close();
  ```