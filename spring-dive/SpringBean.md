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
| Desstruction method      | Bean 销毁回调方法名称                           |

## BeanDefinition 构建

- 通过 BeanDefinitionBuilder  
- 通过 AbstractBeanDefinition 以及派生类