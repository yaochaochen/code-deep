# HashSet、TreeSet源码解析

## HashSet

1. ## 类注释

   看源码先看注释上，可以得到很多信息：

   ```java
   /**
    * This class implements the <tt>Set</tt> interface, backed by a hash table
    * (actually a <tt>HashMap</tt> instance).  It makes no guarantees as to the
    * iteration order of the set; in particular, it does not guarantee that the
    * order will remain constant over time.  This class permits the <tt>null</tt>
    * element.
    *
    * <p>This class offers constant time performance for the basic operations
    * (<tt>add</tt>, <tt>remove</tt>, <tt>contains</tt> and <tt>size</tt>),
    * assuming the hash function disperses the elements properly among the
    * buckets.  Iterating over this set requires time proportional to the sum of
    * the <tt>HashSet</tt> instance's size (the number of elements) plus the
    * "capacity" of the backing <tt>HashMap</tt> instance (the number of
    * buckets).  Thus, it's very important not to set the initial capacity too
    * high (or the load factor too low) if iteration performance is important.
    *
    * <p><strong>Note that this implementation is not synchronized.</strong>
    * If multiple threads access a hash set concurrently, and at least one of
    * the threads modifies the set, it <i>must</i> be synchronized externally.
    * This is typically accomplished by synchronizing on some object that
    * naturally encapsulates the set.
    *
    * If no such object exists, the set should be "wrapped" using the
    * {@link Collections#synchronizedSet Collections.synchronizedSet}
    * method.  This is best done at creation time, to prevent accidental
    * unsynchronized access to the set:<pre>
    *   Set s = Collections.synchronizedSet(new HashSet(...));</pre>
    *
    * <p>The iterators returned by this class's <tt>iterator</tt> method are
    * <i>fail-fast</i>: if the set is modified at any time after the iterator is
    * created, in any way except through the iterator's own <tt>remove</tt>
    * method, the Iterator throws a {@link ConcurrentModificationException}.
    * Thus, in the face of concurrent modification, the iterator fails quickly
    * and cleanly, rather than risking arbitrary, non-deterministic behavior at
    * an undetermined time in the future.
    *
    * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
    * as it is, generally speaking, impossible to make any hard guarantees in the
    * presence of unsynchronized concurrent modification.  Fail-fast iterators
    * throw <tt>ConcurrentModificationException</tt> on a best-effort basis.
    * Therefore, it would be wrong to write a program that depended on this
    * exception for its correctness: <i>the fail-fast behavior of iterators
    * should be used only to detect bugs.</i>
    *
    * <p>This class is a member of the
    * <a href="{@docRoot}/../technotes/guides/collections/index.html">
    * Java Collections Framework</a>.
    *
    * @param <E> the type of elements maintained by this set
    *
    * @author  Josh Bloch
    * @author  Neal Gafter
    * @see     Collection
    * @see     Set
    * @see     TreeSet
    * @see     HashMap
    * @since   1.2
    */
   ```

   1. 底层实现基于HashMap，所有的迭代时不能保证按照顺序插入，或者其他顺序迭代；
   2. add remove contanins size 等方法的耗时性能，不会随着数据量的增加而增加，这个主要和HashMap的底层数组结构有关系，不管数据量多大，都不会导致Hash冲突情况
   3. 线程不安全，如果需要自行加锁或者使用Collections.synchronizeSet;
   4. 迭代过程如果数据结构被修改，会快速失败

### HashSet和HashMap是如何组合的

在类注释中可以知道HashSet实现基于HashMap的，在Java中，要基础类进行创新实现

- 继承基础类，覆盖基础类的方法，比如说继承HashMap，覆盖期add的方法；
- 组合基础类，通过调用其基础类的方法，来复用基础类的能力

### HashSet就是使用组合的HashMap

1. 继承表示父子类属于同一个事物，而Set和Map本来就想表达两种事物，所以继承不妥，而Java语法限制，子类只能继承一个父类，后续很难扩展
2. 组合更灵活，可以任意组合现有的基础类，并且可以在基础类上进行扩展编排等，而方法命名可以任意 无需和基础类的方法名称保持一致。

```java
//把HashMap组合起来，key是Hashset的key，value是PRESENT
private transient HashMap<E,Object> map;
//HashMap的value
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```

从上面代码我们可以看出两点：

1. 使用HashSet时，比如add方法，只需一个入参，但是组合Map的add方法却有key value值，设计的很巧妙，给使用者的体验很好，使用起来简单，可以把底层复杂实现包装一下，默认实现可以自己吃掉，是突出的接口更加简单好用。

### 初始化

HashSet的初始化比较简单，直接new HashMap即可，比较有意思的是，当有原始集合数据，进行初始化，情况，会对HashMap的初始容量进行计算



```java
/**
 * Constructs a new set containing the elements in the specified
 * collection.  The <tt>HashMap</tt> is created with default load factor
 * (0.75) and an initial capacity sufficient to contain the elements in
 * the specified collection.
 *初始化就进行容量计算
 * @param c the collection whose elements are to be placed into this set
 * @throws NullPointerException if the specified collection is null
 */
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
```

上述代码中：Math.max((int) (c.size()/.75f) + 1, 16),就是对HashMap的容量进行了计算，翻译成中文就是取括号的两个数的最大值，(期望值/0.75+1,默认值16) 从计算中，HashSet的实现对HashMap底层实现是非常清楚，

1. 和16比较大小的意思是说，如果给定HashMap初始容量小于16，就按照HashMap的默认16初始化，如果大16就按照给定的初始化。
2. HashMap扩容的伐值的计算公式是Map的容量*0.75f,一旦达到伐值就会扩容，

### 学习设计思想

- 对组合还是继承的分析和把握；
- 对复杂逻辑进行包装，使吐出的接口尽量简单
- 组合其他API时，尽量多对组合的API多一些了解，这样才能更好使用API
- HashMap初始化大小的模板公式

## TreeSet

TreeSet大致的结构和HashSet相似，底层组合的是TreeMap，所以继承了TreeMap key能够实现排序，迭代的时候，也可以按照key的排序顺序进行迭代，主要是复用了TreeMap

### 复用TreeMap的思路一

场景一：TreeSet的add方法

```java
/**
 * Adds the specified element to this set if it is not already present.
 * More formally, adds the specified element {@code e} to this set if
 * the set contains no element {@code e2} such that
 * <tt>(e==null&nbsp;?&nbsp;e2==null&nbsp;:&nbsp;e.equals(e2))</tt>.
 * If this set already contains the element, the call leaves the set
 * unchanged and returns {@code false}.
 *
 * @param e element to be added to this set
 * @return {@code true} if this set did not already contain the specified
 *         element
 * @throws ClassCastException if the specified object cannot be compared
 *         with the elements currently in this set
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 */
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```

底层就是HashMap的put的能力

### 思路二

场景二 需要迭代TreeSet的元素，那应该也像add那样，直接使用HashMap已经有的迭代能力

```java
/**
 * Returns an iterator over the elements in this set in descending order.
 *
 * @return an iterator over the elements in this set in descending order
 * @since 1.6
 */
public Iterator<E> descendingIterator() {
    return m.descendingKeySet().iterator();
}
```

### TreeSet组合TreeMap实现的两种思路

1. TreeSet直接使用TreeMap的某些功能，自己包装新的API
2. TreeSet定义自己想要的API 自己定义接口规范，让TreeMap去实现

方案1和2的调用关系，都是TreeSet调用TreeMap，但功能的实现关系完全相反，第一种是TreeSet的定义和实现都是TreeMap 第二种TreeSet把解耦定义出来，让TreeMap去实现内部逻辑，TreeSet负责接口定义，TreeMap负责具体的实现

### 学习设计思路

1. 像add的方法，直接使用的思路
2. 思路2主要使用复杂的场景，比如迭代场景，TreeSet的场景比较复杂 TreeSet去定义，然后TreeMap去实现

## 面试题

## TreeSet使用场景

一般需要把元素进行排序的时候使用TreeSet，使用时需要最好实现Comparable接口，这样方便根据key进行排序

## 如果实现key的顺序新增进行遍历

按照key的新增顺序进行遍历,LinkedHashMap,LinkedHashSet就是基于HashMap的实现，选择LinkedHashSet

## 实现key的去重

TreeSet的实现是基于TreeMap实现的，TreeMap在put的时候，如果发现key相同就会覆盖value值

