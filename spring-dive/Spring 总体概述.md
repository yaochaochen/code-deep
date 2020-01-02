# Spring 总体概述



## 核心特性（Core）

- Ioc 容器（Ioc Container）
- Spring 事件（Events）
- 资源管理（Resouces）
- 国际化（i18n）
- 校验(Validation)
- 数据绑定（Data Binding）
- 类型转换（Type Conversion）
- Spring 表达式 （Spring Express Language）
- 面向切面（AOP）

## 数据存储(Data Access)

- JDBC 
- 事务抽象（Transaactions）
- DAO 支持（DAO Support）
- O/R映射（O/R Mapping）
- XML 编列（XML Marshalling）

## Web 技术（Web）

- Web Servlet技术栈
  - Spring MVC
  - WebSocket
  - SockJS
- Web Reactive 技术栈
  - Spring WebFlux
  - WebClient
  - WebSocket

## 技术整合（Integration）

- 远程调用（Remoting）
- Java 消息服务（JMS）
- Java 连接架构（JCA）
- Java 管理扩展（JMX）
- Java 邮件客户端（Email）
- 本地任务（Tasks）
- 本地调度（Scheduling）
- 缓存抽象（Caching）
- Spring 测试（Testing）

## 测试（Testing）

- 模拟对象（Mock Objects）
- TestContext框架（TestContext Framework）
- Spring MVC 测试（Spring MVC Test）
- Web 测试客户端（WebTestClient）

## Java 语法变化

- 5（2004）
  - 枚举、泛型、注解、封箱/拆箱等

- 6（2006）
  - `@Override` 接口
- 7（2011）
  - Diamond 语法、多Catch、Try
- 8（2017）
  - Lambda语法、可重复注解、类型注解...
- 9（2017）
  - 模块化、接口私有化方法...
- 10（2018）
  - 局部变量类型推断

## Java 版本依赖于支持

| Spring Framework 版本 | Java标准版本 | Java 企业版本    |
| --------------------- | ------------ | ---------------- |
| 1.X                   | 1.3+         | J2EE1.3+         |
| 2.X                   | 1.4.2+       | J2EE1.3+         |
| 3.X                   | 5+           | J2EE1.4与JavaEE5 |
| 4.X                   | 6+           | Java EE 6 和7    |
| 5.X                   | 8+           | JavaEE 7         |

## Spring 模块化

- ​	spring-aop                                                           spring-jms

- spring-aspects                              						  spring-messaging

  

- spring-context-indexer                                           spring-orm

  

- spring-context-suppot                  				         spring-oxm

  

- spring-context                                                        spring-test

  

- spring-core                                                             spring-tx

  

- spring-expression                                                  spring-web

  

- spring-instrument                                                  spring-webflux

  

- spring-jcl                                                               spring-mvc

  

- spring-jdbc                                                           spring-websoket

