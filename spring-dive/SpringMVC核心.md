# WEB-MVC核心

## 理解Spring Web Mvc架构

### 基础架构servlet

![](/Users/yaochaochen/Desktop/WX20191122-150005.png)

-  	特点
  - ​		请求/响应（Request/Response）
  - 屏蔽网络通讯的细节
- API特性
  - 面向HTTP协议
  - 完整的声明周期
- 职责
  - 处理请求
  - 资源处理
  - 视图渲染

### 核心架构

[前端控制器](http://www.corej2eepatterns.com/FrontController.htm)

![image-20191122150619894](/Users/yaochaochen/Desktop/image-20191122150619894.png)

- 资源：[FrontController](http://www.corej2eepatterns.com/FrontController.htm)
- 实现 Spring Web Mvc [DispatcherServlet](https://docs.spring.io/spring/docs/1.0.0/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)

### 认识Spring Web MVC

#### spring Framework时代的一般认识

##### 实现Controller

```java
@Controller
public class HelloWorldController {
    @RequestMapping("")
    public String index() {
        return "index";
    }
```

##### 配置Web MVC组件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.imooc.web"/>

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

##### 部署DispatcherServlet

```xml
<web-app>

    <servlet>
    <servlet-name>app</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
    <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/app-context.xml</param-value>
    </init-param>
    </servlet>

    <servlet-mapping>
    <servlet-name>app</servlet-name>
    <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

### SpringFrameWork时代的重新认识

