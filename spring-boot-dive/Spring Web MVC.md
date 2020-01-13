# Web  MVC 核心

## 理解Spring Web MVC 架构

### 基础架构

![image-20200113141923587](../../image/image-2020011314191.png)

- 特点
  - 请求/响应(Request/Respose)
  - 屏蔽网络通讯细节
- API 特点
  - 面向 HTTP 协议
  - 完整的生命周期
- 职责
  - 处理资源
  - 资源管理
  - 视图渲染

### 核心架构 [前端控制器 Front Controller](http://www.corej2eepatterns.com/FrontController.htm)

![image-20200113143543750](../../image/image-1.png)



![image-20200113143619093](../../image/image-2.png)

- 资源 http://www.corej2eepatterns.com/FrontController.htm
- 实现: Spring Web MVC DispatcherServlet
  -  https://docs.spring.io/spring/docs/1.0.0/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html

### Spring Web MVC 架构

![image-20200113143805930](../../image/image-3.png)

### 认识 Spring Web MVC

#### Spring Framework 时代的一般认识

##### 实现Controller

```java
@Controller public class HelloWorldController { 
  @RequestMapping("") 
  public String index() {
  return "index"; 
  } 
}
```

#### 配置 Web MVC

```xml
<?xml version="1.0" encoding="UTF-8"?> <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"> <context:component-scan base-package="com.imooc.web"/> 
<bean 		class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/> <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/> <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver"> <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/> <property name="prefix" value="/WEB-INF/jsp/"/> <property name="suffix" value=".jsp"/> </bean> 
</beans>
```

### Spring Framework 时代的重新认识

#### Web MVC 核心组件

|                        组件 Bean 类型                        | 说明                                                         |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
| [HandlerMapping](https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/web.html#mvc-handlermapping) | 映射请求Request到处理器 Handler 加上其关联的拦截器 HandlerInterceptor 列表，其映射关系基于不同的 HandlerMapping 实现一些标准细节 其中主要HandlerMapping 实现 RequestMappingHandlerMapping 支持标注@RequestMapping 的方法 SimpleUrlHandlerMapping 维护精准的URI 路劲与处理器的映射。 |
|                       HandlerAdapater                        | 帮助 DispatcherServlet  调用请求处理器 Handler 无需关注其中实际的调用细节 比如 调用注解实现的 Controller 需要解析其关联的注解 HandlerAdapter 的主要目标就是屏蔽 DispatcherServlet 之间的实现细节 |
| [ExceptionHandlersResolver](https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers) | 解析异常 可能策略是将异常处理映射到其他处理器 (Hanlers) 或者 某个 HTTP 错误页面 |
| [ViewreSolver](https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/web.html#mvc-viewresolver) | 从处理器 Handler 返回字符类型的逻辑视图名称解析出来实际的 View 对象 该对象将渲染后的内容输出到 HTTP 响应中 |
| [LocalResolve](https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/web.html#mvc-localeresolver) | 从客户端解析 Locale 为其实现国际化视图                       |
| [MultipartResolve](https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/web.html#mvc-multipart) | 解析多部分请求 文件上传的抽象实现                            |

#### 交互流程

![image-20200113151035601](../../image/image-4.png)

### Web MVC 注解驱动

- 版本依赖

  - Spring Framework 3.1 +

  ##### 基本配置步骤

  注解配置 @Configuration （Spring 范式注解）

  组件激活 @EnableWebMvc （Spring 模块装配）

  自定义组件 WebMvcConfigurer (Spring  Bean)

##### 常用注解

注册模型属性 @ModelAttribute

读取请求头   @RequestHearder

读取 Cookie @CookieValue

校验处理 @Valid @Validdated

注解处理 @ExceptionHandler

切面通知 @ControllerAdvice

##### 自动装配

- 版本依赖

  - Spring Framework 3.1 +
  - Servlet 3.0 +

  Servlet SPI

  Servlet SPI ServletContainerInitializer ，参考 Servlet 3.0 规范

  配合 @HandlerTypes

  

  ##### Spring SPI

  基础接口: WebApplicationInitializer

  编程驱动: AbstractDispatcherServletInitializer

  注解驱动:AbstractAnnotationConfigDispatcherServletInitializer

#### 简化 Spring Web MVC 

#### Spring Boot 时代的简化

##### 	全完自动装配

​	自动装配：DispatcherServlet: DispatcherServletAutoConfiguration

​		

​		

