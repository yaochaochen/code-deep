# Spring Bean 作用域

1. Spring Bean 的作用域

   | 来源        | 说明                                                       |
   | ----------- | ---------------------------------------------------------- |
   | singleton   | 默认 Spring Bean 作用域，一个 BeanFactory 有且仅有一个实例 |
   | prototype   | 原型作用域 每次依赖查找和依赖注入生成新 Bean对象           |
   | request     | 将 Spring Bean 储存在 ServletRequest 上下文中              |
   | session     | 将 Spring Bean 储存在 HttpSession 中                       |
   | application | 将 Spring Bean 储存在 ServletContext 中                    |

   

2. singleton Bean 作用域

   - 配置

     ```xml
     <bean id = "", class = "">
     	<property name = "" ref = ""/>
     </bean>
     ```

     在全局范围是共享的单例 在当前ClassLoader操作 如果ClassLoader 对应上下文 通过id进行关联

     判断 Bean 是不是单例 在 BeanDefintion 元信息有没有 isSingleton 

3. prototype Bean 作用域

   - 配置

     ```xml
     <bean id = "", class = "">
     	<property name = "" ref = "" scope="property"/>
     </bean>
     ```

     产生新的 Bean 对象

4. request 作用域

5. session Bean作用域

6. Application Bean 作用域

7. 自定义 Bean 作用域

