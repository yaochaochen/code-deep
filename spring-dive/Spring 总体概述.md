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

## Spring 对 java 语言的运用

- ### Java 5 语法特性

  | 语法特性             | Spring 支持版本 | 代表实现                   |
  | -------------------- | --------------- | -------------------------- |
  | 注解(Annotation)     | 1.2+            | @Transactional             |
  | 枚举(Enumeration)    | 1.2+            | Propagation                |
  | for-each 语法        | 3.0+            | AbstractApplicationContext |
  | 自动装箱(AutoBoxing) | 3.0+            |                            |
  | 泛型(Generic)        | 3.0+            | AppliactionListenner       |

  - ### Java 6 语法特性

  | 语法特性         | Spring 支持版本 | 代表实现 |
  | ---------------- | --------------- | -------- |
  | 接口 `@Override` | 4.0             |          |

  - ### Java 7 语法特性

  | 语法特性               | Spring支持版本 | 代表实现                    |
  | ---------------------- | -------------- | --------------------------- |
  | Diamond语法            | 5.0+           | DefaultListableBeanFactory  |
  | try-with-resources语法 | 5.0+           | ResourceBundleMessageSource |

  ###  Java 8 语法特性

  | 语法特性 | Spring支持版本 | 代表实现                     |
  | -------- | -------------- | ---------------------------- |
  | Lambda   | 5.0+           | PropertyEditorRegistrySuppot |

  ## Spring 对 JDK API 的实现

  #### JDK 核心 API

  #####  

  ##### <Java 5

  反射(Reflection)

  Java Beans

  动态代理(Dynamic Proxy)

  ##### Java 5

  并发框架(J.U.C)

  格式化(Formatter)

  Java管理扩展(JMX)

  Instrumentation

  XML 处理(DOM SAX XPath XSTL)

  ##### Java 6

  JDBC 4.0(JSR 221)

  JAXB 2.0(JSR 222)

  可插拔注解处理API(JSR 269)

  Common Annotations(JSR 250)

  Java Compiler API (JSR 199)

  Scripting JVM(JSR 223)

  ##### Java7 

  NIO 2(JSR 203)

  Fork/Join框架(JSR 166)

  Invokedynamic 字节码(JSR 292)

  ##### Java 8 

  Stream API (JSR 335)

  CompletableFuture(J.U.C)

  Annotation on java Types(JSR 308)

  Date and Time API(JSR 310)

  可重复 Annontion(JSR 337)

  JavaScript 运行时(JSR 223)

  ##### Java 9

  Reactive Streams Flow API(J.U.C)

  Process API Updates(JEP 102)

  Variable Handles(JEP 227)

  Method Handler(JEP 277)

  Spin-Wait Hints(JEP 285)

  Stack-Walking API (JEP 259)

  