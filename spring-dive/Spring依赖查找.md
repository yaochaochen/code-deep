# Spring IoC 依赖查找

## 依赖查找的今世前生

## 单一类型依赖查找

-  根据 Bean 名称查找
  - getBean(String)
  - Spring 2.5 覆盖默认参数: getBean(String, Object...)
- 根据 Bean 类型查找
  - Bean实时查找
    - Spring 3.0 getBean(Class)
    - Spring4.1覆盖默认参数:getBean(Class, Object...)
  - Spring5.1 Bean 延迟查找
    - getBeanProvider(Class)
    - getBeanProvider(ResolvableType)
- 根据 Bean 名称 + 类型查找 getBean(String, Class) 

## 集合类型依赖查找

- 集合类型依赖查找接口 -ListableBeanFactory
  - 根据 Bean 类型查找
    - 获取同类型 Bean 名称列表
      - getBeanNamesForType(Class)
      - Spring 4.2 getBeanNamesForType(ResolvableType)
    - 获取同类型 Bean 实例列表
      - getBeansOfType(Class)以及重载方法
- 通过注解类型查找
  - Spring 3.0 获取标注类型 Bean 名称方法列表
    - getBeanNamesForAnnotation(Class<? extends Annatation>)
  - Spring 3.0 获取标注类型 Bean 实例列表
    - getBeansWithAnnotation(Class<? extends Annotation>)
  - Spring 3.0 获取指定名称 + 标注类型 Bean 实例
    - findAnnotationOnBean(String, Class<? extends Annotation>)

## 层次性依赖查找

## 延迟依赖查找

## 安全依赖查找

## 内建可查找的依赖

## 依赖查找中的经典异常

## 

