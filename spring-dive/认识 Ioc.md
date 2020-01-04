# 认识 IoC

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