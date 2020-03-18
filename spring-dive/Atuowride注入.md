# @Autowried 注入

- Atuowried 注入过程

  - 元信息解析

    元信息解析过程中有一个类DependencyDescriptor 依赖描述器 把相关信息加入进去

    元信息包含 Bean 、Bean字段、filed字段

    通过 Java 反射 进行 Bean 的注入

  - 依赖查找

  - 依赖注入（字段、方法）

- 自己的一点总结：
  1.在doCreateBean中会先调用applyMergedBeanDefinitionPostProcessors，后执行populateBean
  所以会先调用postProcessMergedBeanDefinition后执行InstantiationAwareBeanPostProcessor的postProcessProperties。
  2.postProcessProperties中有两个步骤：
  （1）findAutowiringMetadata查找注入元数据，没有缓存就创建，具体是上一节内容。最终会返回InjectionMetadata，里面包括待注入的InjectedElement信息（field、method）等等
  （2）执行InjectionMetadata的inject方法，具体为AutowiredFieldElement和AutowiredMethodElement的Inject方法
  （2.1）AutowiredFieldElement inject具体流程：
  （2.1.1）DependencyDescriptor的创建
  （2.1.2）调用beanFactory的resolveDependency获取带注入的bean
  （2.1.2.1）resolveDependency根据具体类型返回候选bean的集合或primary 的bean
  （2.1.3）利用反射设置field

- @ Inject 注入
  - 如果 JSR-330 存在于 ClassPath 中，复用 AtuowiredAnntationBeanPostProcessor 实现
- Java 通用注解原理
  - CommonAnnotationBeanPOSTProcessor
    - 注入注解
      - javax.xml.ws.WebServiceRef
      - java.ejb.EJB
      - javax.annontation,Resource
    - 生成周期注解
    - javax.annotation.PostConstruct
    - javax.annotation.PreDestory

- 面试题

  - 有多少种依赖注入方式

    - 构造器注入
    - Setter 注入
    - 字段注入
    - 方法注入
    - 接口回调注入

  - 你偏好构造器注入还是 Setter 注入

    - 两种依赖注入均可使用 如果必须依赖的话，那么推荐使用构造器注入， Setter 注入用于可选依赖

  - Spring 依赖注入的来源有那些

    

