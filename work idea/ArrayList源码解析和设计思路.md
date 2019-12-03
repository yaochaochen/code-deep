## ArrayList源码解析和设计思路

### 整体架构

数组结构

| 0    | 1    | 2    |
| ---- | ---- | ---- |
| o1   | o2   | o3   |

第一行表示数组下标index，从0开始计数

第二行表示数组本身elementData从1开始

### ArrayList定义的常量含义

```java
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;
```

- 表示数组初始大小默认是10

```java
 /* The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;
```

- size表示当前的大小，类型是int，没有volatile修饰，非线程安全的。

```java
protected transient int modCount = 0;
```

- modCount统计当前数组被修改的版本次数，数组结构有变动，就会+1

### 源代码类头部注释解释

- 允许put null 值， 会自动扩容

- size isEmpty get set add 等方法的复杂度是O(1)

- 是非线程安全的，多线情况下，推荐使用线程安全类: Collections@sysnchronizedList

- 增强for循环，或者使用迭代器过程中如果代码数组大小被改变，会快速失败，抛出异常。

  ### ArrayList的构造方法

  - 无参数的初始化

    ```java
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    ```

  -  指定初始数据的初始化

    ```java
    public ArrayList(Collection<? extends E> c) {
      //默认为null
      //保存到elementData的数组容器
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
              //强制转换为Object类型
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
    ```

  - 指定大小的初始化

    ```java
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    ```

  - ArrayList无参构造初始化时，默认大小是空数组，并不是大家常说的是10，其实10是第一次add的时候进行数组扩容的大小。

  - 指定初始数据的初始化时，存在一个bug 意思是指定集合内的元素不是Object类型时，会转化Object。在特殊场景里会触发bug ArrayList不是Object 调用toArray()得到Object数组 往Object赋值时，会触发bug。在Java9里已经解决这个问题了。

    ```java
    List<String> list = Arrays.asList("hello, world");
    Object[] objects = list.toArray();
    System.out.println(objects.getClass().getSimpleName());
    objects[0] = new Object();
    String[]
    Exception in thread "main" java.lang.ArrayStoreException: java.lang.Object
    	at com.yuetu.deep.in.java.collection.ArrayListDemo.main(ArrayListDemo.java:15)
    ```

    ### 新增和扩容实现

    #### 新增就是往数组里添加元素，主要分两步

    - 判断是否需要扩容 如果需要需要执行扩容操作

    - 直接赋值

      #### 代码实现

      ```java
      public boolean add(E e) {
        //确保数组大小是否足够，不够执行扩容，size为当前大小
          ensureCapacityInternal(size + 1);  // Increments modCount!!
          //直接赋值，线程不安全的
        	elementData[size++] = e;
          return true;
      }
      ```

      ##### ensureCapacityInternal 扩容机制

      ```java
      private void ensureCapacityInternal(int minCapacity) {
        //如果初始化数组大小时，有给定初始值 以给定的大小为准，不走if逻辑
          ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
      }
      //确保扩容体积足够
      private void ensureExplicitCapacity(int minCapacity) {
        //确保数组被修改
          modCount++;
      // 如果我们期望的最小容量大于目前数组的长度，那么就扩容
          // overflow-conscious code
          if (minCapacity - elementData.length > 0)
              grow(minCapacity);
      }
      ```

      ```java
      private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
      
      /**
       * Increases the capacity to ensure that it can hold at least the
       * number of elements specified by the minimum capacity argument.
       *扩容，并把现有数据拷贝到新的数组里面去
       * @param minCapacity the desired minimum capacity
       */
      private void grow(int minCapacity) {
          // overflow-conscious code
          // 如果扩容后的值 < 我们的期望值，扩容后的值就等于我们的期望值
          int oldCapacity = elementData.length;
          int newCapacity = oldCapacity + (oldCapacity >> 1);
         // 如果扩容后的值 < 我们的期望值，扩容后的值就等于我们的期望值
          if (newCapacity - minCapacity < 0)
              newCapacity = minCapacity;
         // 如果扩容后的值 > jvm 所能分配的数组的最大值，那么就用 Integer 的最大值
          if (newCapacity - MAX_ARRAY_SIZE > 0)
              newCapacity = hugeCapacity(minCapacity);
          // minCapacity is usually close to size, so this is a win:
          elementData = Arrays.copyOf(elementData, newCapacity);
      }
      
      private static int hugeCapacity(int minCapacity) {
          if (minCapacity < 0) // overflow
              throw new OutOfMemoryError();
          return (minCapacity > MAX_ARRAY_SIZE) ?
              Integer.MAX_VALUE :
              MAX_ARRAY_SIZE;
      }
      ```

    扩容本质

    1. 扩容的规则并不是翻倍，是原来容量大小+容量大小的一半，其实就是1.5倍
    2. ArrayList数组的最大值是Integer.MAX_VALUE，超过这个值的话，JVM就不会给数据分配内存空间了
    3. 新增时，并没有对值进行严格的数据校验，所以可以允许为null的值
    4. 从扩容机制上可以看到，ArrayList对数组大小溢出问题的意识存在的
    5. 扩容后就直接赋值了。没有任何锁机制，所以这里的所有方法操作都存在线程安全问题

    ### 扩容的实现

    `Arrays.copyOf(elementData, newCapacity);`

    这行代码描述的本质就是数组的拷贝 扩容是新建一个符合预期容量的新数组，然后把老数组的数据copy过去

    通过System.arraycopy()进行copy 到这里 ArrayList就走到JVM底层实现了，

    ##### 删除

    ​	ArrayList有很多种删除方式，比如根据数组索引删除，根据值删除或者批量删除...

    - 根据值删除

    ​	

    ```java
    public boolean remove(Object o) {
      //如果值为空 找到第一个值为null的删除
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
          //如果值不为null 找到第一个要删除的值相等的删除
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    ```

    1. 在新增的时候，没有对null进行校验 所以删除的时候是允许删除null值的
    2. 找到值在数组中的索引位置，是通过equals来判断，如果数组元素不是基本类型，需要关注equals的具体实现

    - 按照索引位置删除

    ```java
    private void fastRemove(int index) {
      //记录数组发生变化
        modCount++;
      //删除index位置的元素后，需要移动剩余的元素位置，-1是因为index从0开始算起，size从1开始
        int numMoved = size - index - 1;
        if (numMoved > 0)
          //index+1位置开始copy 长度是numMoved
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
      //最后一位赋值为null
        elementData[--size] = null; // clear to let GC do its work
    }
    ```

    ### 迭代器

    - hashNext还有没有值可以迭代
    - next 如果有值可以迭代 迭代值是多少
    - remove 删除当前迭代的值

    ### 时间复杂度

    对新增和删除的方法来看，ArrayList其实是对数组元素操作的，只需要数组索引，直接新增和删除，所有时间复杂度是O(1)

    ### 线程安全

    只有ArrayList作为共享变量时，才会存在线程不安全的情况，当ArrayList方法对内部的局部变量时，是没有线程安全的问题

    ArrayList线程不安全的本质来源是 elementData、size、modCount在进行各种操作时，都没有加锁，而且这些变量都是不可见的（volatile） 如果多线程操作可能覆盖原来的值。

    使用Collections#synchridizedList来保证线程安全 但是只是对每个方法进行加入轻量锁来实现的，对性能大大降低。

    ### 总结

    ArrayList其实就是数组操作进行封装API而已

    