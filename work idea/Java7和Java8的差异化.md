# Java7和Java8的差异化

### 概述

Java8在Java7的基础上，做了一些改进和优化，在改动是如何实现兼容Java老版本，Java8又如何改进的？

### 通用

1. 所有集合都新增了，forEach方法

   List、Set、Map在Java8版本中都新增了forEach的方法，方法入参都是Consumer，Consumer是一个函数式接口，可以简单理解允许一个入参，但没有返回值的函数接口，我们以ArrayList的forEach的源码为例，

   ```java
   @Override
   public void forEach(Consumer<? super E> action) {
     //判断是否空
       Objects.requireNonNull(action);
     //原始值被拷贝
       final int expectedModCount = modCount;
       @SuppressWarnings("unchecked")
     	final E[] elementData = (E[]) this.elementData;
       //每次循环前检测是否被修改，一旦修改会停止的
     	final int size = this.size;
       for (int i=0; modCount == expectedModCount && i < size; i++) {
           action.accept(elementData[i]);//这玩意是啥？
       }
     	//数组如果被修改，抛异常
       if (modCount != expectedModCount) {
           throw new ConcurrentModificationException();
       }
   }
   ```

   -  action.accept(elementData[i]) 就是在forEach中操作任何事件
   - @Override 说明可以被继承实现，改方法被定义到Iterable接口上，Java7和Java8都实现了该接口，但是我们在Java7和Java8的ArrayList都实现了该接口，但我们在Java7和Java8中，都实现了ArrayList接口，但是Java7的ArrayList并没有实现该方法，编译器也没有报错 这个主要因为forEach方法是被default修饰的。

   在Set Map 都是通过default修饰的。

   ### List区别

   #### ArrayList

   ArrayList无参初始化时，Java7是直接初始化10的大小，Java8去掉了这个逻辑，初始化是个空数组，在第一次add时，才开始按照10进行扩容

   ### Map

   #### HashMap

   1. 和ArrayList一样，Java8中的HashMap在无参构中，丢弃了Java7中直接把数组初始化16的做法，而采用了第一次新增时，进行扩容的
   2. Hash算法计算公式不同，Java8的Hash算法更加简单，代码更加简介
   3. Java8的HashMap增加了红黑树的数据结构，这个是Java7么有的，java7中只有链表+数组的结构

[http://openjdk.java.net/projects/jdk8/features#103%E3%80%82]: 	"java8新增功能"

