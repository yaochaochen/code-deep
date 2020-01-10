# Spring IoC 容器概述

## IoC 包含的内容

1. Spring IoC 依赖查找
2. Spring IoC 依赖注入
3. Spring IoC 依赖源
4. Spring IoC 配置源信息
5. Spring IoC 容器
6. Spring 应用上下文
7. 使用Spring IoC 容器
8. Spring IoC 容器生命周期

## IoC 依赖查找

- 根据Bean名称查找
  - 延时查找
  
    ```java
    private static void lookupLazy(BeanFactory beanFactory) {
      ObjectFactory<User> userObjectFactory = (ObjectFactory<User>) beanFactory.getBean("objectFactory");
      User user = userObjectFactory.getObject();
        System.out.println("延时查找" + user);
    }
    ```
  
  - 实时查找
  
  - ```java
    private static void lookupRealTime(BeanFactory beanFactory) {
        User user =  (User) beanFactory.getBean("user");
        System.out.println("实时查找" + user);
    }
    ```
  
- 根据 Bean 类型查找
  - 单个 Bean 对象
  
  - 集合 Bean 对象
  
    ```java
    if (beanFactory instanceof ListableBeanFactory) {
        ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
        Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
        System.out.println("查找所有的 User 集合对象" + users);
    }
    ```
  
    
  
- 根据 Bean 名称 + 类型查找

- 根据 Java 注解查找
  - 单个 Bean 对象
  - 集合 Bean 对象 
  
  ```java
  if (beanFactory instanceof ListableBeanFactory) {
      ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
      Map<String, User> users = (Map) listableBeanFactory.getBeansWithAnnotation(Super.class);
      System.out.println("查找所有的被 Super 标注的集合对象" + users);
  }
  ```
  
  ## IoC 依赖来源
  
  - 自定义 Bean
  - 容器内建 Bean 对象
  - 容器内建依赖
  
  ```java
  ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:/META-INF/dependency-injection-context.xml");
  BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath:/META-INF/dependency-injection-context.xml");
  //依赖来源-:自定义 Bean
  UserRepository userRepository = applicationContext.getBean("userRepository", UserRepository.class);
  System.out.println(userRepository.getUsers());
  
  //依赖来源二 依赖注入 内建依赖
  System.out.println(userRepository.getObjectFactory());
  
  //容器内建  Bean
  Environment environment =  beanFactory.getBean(Environment.class);
  System.out.println(environment.toString());
  ```

## IoC 配置源信息

- Bean 定义配置
  - 基于 XML 文件
  - 基于 Properties 文件
  - 基于 Java 注解
  - 基于 Java API
- IoC 容器配置信息
  - 基于 XML 文件
  - 基于注解
  - 基于 Java API
- 外部化属性配置
  - 基于 Java 注解

## Spring IoC 配置源



## Spring 应用上下文