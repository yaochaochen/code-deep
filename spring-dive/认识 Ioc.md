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

