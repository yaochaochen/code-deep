# 认识 IoC



## IoC 容器的职责

- 通用职责
- 依赖处理
  - 依赖查找(主动行为)
  - 依赖注入
- 声明周期管理
  - 容器
  - 托管的资源（Java Beans 或者其他资源）
- 配置
  - 容器
  - 外部化配置
  - 托管的资源（Java Beans 或者其他资源）

## IoC 容器的实现

- ### 主要实现

  - Java SE
    - Java Beans
    - Java ServiceLoader SPI 
    - JNDI Java Naaming and Directory Interface
  - Java EE
    - EJB Enterprice Java Beans
    - Servlet
  - 开源
    - Apache Avalon https://avalon.apache.org/closed.html
    - PiocContainer http://picocontainer.com/
    - Google Guice https://github.com/google/guice
    - Spring Framework https://spring.io/projects/spring-framework

## 传统 IoC 容器的实现

- Java Beans 作为 IoC 容器
- 特性
  - 依赖查找
  - 生命周期管理
  - 配置元信息
  - 事件
  - 自定义
  - 资源管理
  - 持久化
- 规范
  - Java Beans https://docs.oracle.com/javase/8/docs/api/index.html
  - BeanContext https://docs.oracle.com/javase/8/docs/api/index.html

## 轻量级 IoC 容器

- 容器管理代码
- 快速启动
- 容器不需要特殊的配置



## 依赖查找 VS 依赖注入

- 优劣势对比

| 类型     | 依赖处理 | 实现便利性 | 代码侵入性   | API 依赖性   | 可读性 |
| -------- | -------- | ---------- | ------------ | ------------ | ------ |
| 依赖查找 | 主动获取 | 相对繁琐   | 侵入业务逻辑 | 依赖容器 API | 良好   |
| 依赖注入 | 被动提供 | 相对便利   | 低侵入性     | 依赖容器 API | 一般   |

## 构造器注入 VS Setter 注入

- ​	Spring Framework 对构造器注入与 Setter 的论点:

> The Spring team generally advocates constructor injection as it enables one to implement application components as immutable objects and to ensure that required dependencies are not null. Furthermore constructor-injected components are always returned to client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.
>
> Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later. Management through JMX MBeans is therefore a compelling use case for setter injection.
>
> Use the DI style that makes the most sense for a particular class. Sometimes, when dealing with third-party classes for which you do not have the source, the choice is made for you. For example, if a third-party class does not expose any setter methods, then constructor injection may be the only available form of DI.

​		如果依赖注入为空的话，可以使用另外的方式进行注入，`ObjectProvider` 这个类 是一种类型安全的，如果类型是简单类型调用 `getIfAvailable`方法

Advantages of Setter Injection include:

> - ❑  JavaBean properties are well supported in IDEs.
> - ❑  JavaBean properties are self-documenting.
> - ❑  JavaBean properties are inherited by subclasses without the need for any code.
> - ❑  It’s possible to use the standard JavaBeans property-editor machinery for type conversions if necessary.
> - ❑  Many existing JavaBeans can be used within a JavaBean-oriented IoC container without modifi- cation. For example, Spring users often use the Jakarta Commons DBCP DataSource. This can be managed via its JavaBean properties in a Spring container.
> - ❑  If there is a corresponding getter for each setter (making the property readable, as well as writable), it is possible to ask the component for its current configuration state. This is particu- larly useful if we want to persist that state: for example, in an XML form or in a database. With Constructor Injection, there’s no way to find the current state.
> - ❑  Setter Injection works well for objects that have default values, meaning that not all properties need to be supplied at runtime.

## 	面试IOC

### 		什么是IOC

​			简单的说， IOC是控制反转 类似好莱坞原则，主要依赖查找和依赖注入的实现

### 		依赖查找和依赖注入

​	依赖查找是主动或者手动的依赖查找方式，通常是通过容器API或者标准，API的实现 而依赖注入则是依赖绑定方式，无需容器API

### Spring IOC有什么优势

- 依赖查找和依赖注入
- AOP抽象
- 事务抽象
- 事件机制
- SPI 扩展
- 强大的第三方整合
- 更好的面向对象

## 补充说明

维基百科(https://en.wikipedia.org/wiki/Inversion_of_control#Overview)

Inversion of control carries the strong connotation that the reusable code and the problem-specific code are developed independently even though they operate together in an application. [Software frameworks](https://en.wikipedia.org/wiki/Software_framework), [callbacks](https://en.wikipedia.org/wiki/Callback_(computer_programming)), [schedulers](https://en.wikipedia.org/wiki/Scheduling_(computing)), [event loops](https://en.wikipedia.org/wiki/Event_loop), [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection), and the [template method](https://en.wikipedia.org/wiki/Template_method) are examples of [design patterns](https://en.wikipedia.org/wiki/Design_pattern) that follow the inversion of control principle, although the term is most commonly used in the context of [object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming).

Inversion of control serves the following design purposes:

- To [decouple](https://en.wikipedia.org/wiki/Object_decoupling) the execution of a task from implementation.
- To focus a module on the task it is designed for.
- To free modules from assumptions about how other systems do what they do and instead rely on [contracts](https://en.wikipedia.org/wiki/Design_by_contract).
- To prevent [side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) when replacing a module.

Inversion of control is sometimes facetiously referred to as the "Hollywood Principle: Don't call us, we'll call you".