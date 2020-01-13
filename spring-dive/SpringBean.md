# Spring Bean 

## Spring Bean 基础

1. 定义 Spring Bean
2. BeanDefinition 元信息
3. 命名 Spring Bean
4. Spring Bean 别名
5. 注册 Spring Bean
6. 实例化 Spring Bean
7. 初始化 Spring Bean
8. 延迟初始化 Spring Bean
9. 销毁 Spring Bean
10. 垃圾回收Spring Bean

## 定义 Spring Bean

- 什么是 BeanDefinition
- BeanDefinition 是Spring Framework 中定义 Bean 的配置元信息接口，包含:
  - Bean 的类名
  - Bean 行为配置元素 作用域 自动绑定的模式 生命周期回调等
  - 其他 Bean 引用 
  - 配置设置

## BeanDefinition 元信息

| 属性(Property)           | 说明                                            |
| ------------------------ | ----------------------------------------------- |
| Class                    | Bean 全类名，必须是具体的类，不能抽象类或者接口 |
| Name                     | Bean 的名称或者ID                               |
| Scope                    | Bean 的作用域                                   |
| Constructor arguments    | Bean 构造器参数 用于依赖注入                    |
| Properties               | Bean 属性设置 用于依赖注入                      |
| Autowiring mode          | Bean 自动绑定模式                               |
| Lazy initialization mode | Bean 延迟加载初始化模式                         |
| Initalization method     | Bean 初始化回调方法名称                         |
| Destruction method       | Bean 销毁回调方法名称                           |

## BeanDefinition 构建

- 通过 BeanDefinitionBuilder  

  ```java
   BeanDefinitionBuilder builder =  BeanDefinitionBuilder.genericBeanDefinition(User.class);
          //设置属性
          builder.addPropertyValue("age", 43);
          builder.addPropertyValue("name", "Spring-Dive");
  ```

  

- 通过 AbstractBeanDefinition 以及派生类

```java
 GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
        genericBeanDefinition.setBeanClass(User.class);
        MutablePropertyValues mutablePropertyValues = new MutablePropertyValues();
        mutablePropertyValues.addPropertyValue("age", 1);
        mutablePropertyValues.addPropertyValue("name", "Spring");
        genericBeanDefinition.setPropertyValues(mutablePropertyValues);
```

##  命名 Spring Bean

- Bean 名称

  每个 Bean 拥有一个或多个标记符（identifiers）, 这些标识在 Bean 所在的容器必须是唯一的，通常，一个Bean仅有一个标识符 如果需要额外的，可以考虑使用别名（Alias）来扩充。

  ​	在基于 XML 的配置元信息中，开发人员可以通过 id 或者 name 属性规范 Bean  的标识符。通常 Bean 的标识符有字母组成， 允许出现特殊字符，如果想引入 Bean 的别名的话，可以 name 属性使用 半角符号 或者分号 来分隔。

  ​	Bean 的 id 和 name 属性非必须制定，如果留空的话，容器会为 Bean 自动生成一个唯一的名称。 Bean的命名尽管没有限制，不过官方建议采取驼峰命名

- Bean 名称生成器（BeanNameGenerator）

- 由 Spring  Framework2.0.3 引入 框架内建两种实现

  ​	DefaultBeanNameGenerator 默认通用 BeanNameGenerator 实现的

- AnnotationBeanNameGenerator: 基于注解扫描 BeanNameGenerator 实现，起始于Spring Framework 2.5