# Spring IoC 依赖查找

## 依赖查找的今世前生

## 单一类型依赖查找

-  根据 Bean 名称查找
  - getBean(String)
  - Spring 2.5 覆盖默认参数: getBean(String, Object...)
- 根据 Bean 类型查找
  - Bean实时查找
    - Spring 3.0 getBean(Class)
    - Spring4.1覆盖默认参数:getBean(Class, Object...)
  - Spring5.1 Bean 延迟查找
    - getBeanProvider(Class)
    - getBeanProvider(ResolvableType)
- 根据 Bean 名称 + 类型查找 getBean(String, Class) 

## 集合类型依赖查找

- 集合类型依赖查找接口 -ListableBeanFactory
  - 根据 Bean 类型查找
    - 获取同类型 Bean 名称列表
      - getBeanNamesForType(Class)
      - Spring 4.2 getBeanNamesForType(ResolvableType)
    - 获取同类型 Bean 实例列表
      - getBeansOfType(Class)以及重载方法
- 通过注解类型查找
  - Spring 3.0 获取标注类型 Bean 名称方法列表
    - getBeanNamesForAnnotation(Class<? extends Annatation>)
  - Spring 3.0 获取标注类型 Bean 实例列表
    - getBeansWithAnnotation(Class<? extends Annotation>)
  - Spring 3.0 获取指定名称 + 标注类型 Bean 实例
    - findAnnotationOnBean(String, Class<? extends Annotation>)

## 层次性依赖查找

- 层次性依赖查找接口 -HierarchicalBeanFactory
  - 双亲 BeanFactory：getParentBeanFactory()
  - 层次性查找
    - 根据 Bean 名称查找
      - 基于containsLocalBean 方法
    - 根据 Bean 类型查找实例列表
      - 单一类型 BeanFactoryUtils#beanOfType
      - 集合类型 BeanFactoryUtils#beansOfTypeIncludingAncestors
    - 根据 Java 注解查找名称列表
      - BeanFactoryUtils#beanNamesForTypeIncludingAncestors



## 延迟依赖查找

- Bean 延迟依赖查找接口

  - ```java
    org.springframework.beans.factory.ObjectFactory
    ```

    

    - ```java
      org.springframework.beans.factory.ObjectProvider
      ```

      - Spring 5 对 Java 8 特性扩展
        - 函数式接口
          - getlfAvailable(Supplier)
          - ifAvailable(Consumer)
        - Stream 扩展 -stream()

    

## 安全依赖查找

- 依赖查找安全性对比

  | 依赖查找类型 | 代表实现                           | 是否安全 |
  | ------------ | ---------------------------------- | -------- |
  | 单一类型查找 | BeanFactory#getBean                | 否       |
  |              | ObjectFactory#getObject            | 否       |
  |              | ObjectProvider#getlfAvailable      | 是       |
  | 集合类型查找 | ListableBeanFactory#getBeansOfType | 是       |
  |              | ObjectProvider#stream              | 是       |

   **注意:  层次性依赖查找的安全性取决于其扩展的单一或集合类型的 BeanFactory**

## 内建可查找的依赖

- AbstractAPPlicationContext 内建可查找的依赖

| Bean 名称                   | Bean 实例                        | 使用场景              |
| --------------------------- | -------------------------------- | --------------------- |
| environment                 | Environment 对象                 | 外部化配置            |
| systemProperties            | java.util.Properties 对象        | Java 系统属性         |
| systemEnvironment           | java.util.Map 对象               | 操作系统环境变量      |
| messageProcessor            | LifecycleProcessor 对象          | Lifecycle Bean 处理器 |
| ApplicationEventMulticaster | ApplicationEventMulticaster 对象 | Spring 事件广播器     |



## 依赖查找中的经典异常

## 

