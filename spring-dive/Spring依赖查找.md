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

## 层次性依赖查找

## 延迟依赖查找

## 安全依赖查找

## 内建可查找的依赖

## 依赖查找中的经典异常

## 

