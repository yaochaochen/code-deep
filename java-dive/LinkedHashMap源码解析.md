# 					LinkedMap源码解析

## 为什么会出现LinkedMap

HashMap本身是无序的，TreeMap是实现根据key排序，Map就会必然能插入有顺序的的链表，那就是LinkedMap的实现来完成有序的链表

所谓有序插入顺序进行访问。

实现访问最少最先删除功能

## 常量

```java
/**
 * The head (eldest) of the doubly linked list.
 */
//链表头部
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
//链表尾部
transient LinkedHashMap.Entry<K,V> tail;

/**
 * The iteration ordering method for this linked hash map: <tt>true</tt>
 * for access-order, <tt>false</tt> for insertion-order.
 *
 * @serial
 */
//访问模式 默认false 按照插入顺序提供访问
//true按照访问顺序，会把经常访问的key放在尾部
//
final boolean accessOrder;
```

上述常量可以看出来，LinkedHashMap的数据结构就像把LinkedList的每个元素换成HashMap的Node，两者结合起来，正因为这增加了这种数据结构，才把Map元素串联起来的，形成链表，而链表保证了有序，就能维护有序插入

### 顺序新增

LinkedHashMap初始化，默认AccessOrder为false，就会按照插入顺序提供访问，插入方法是父类的HashMap的put方法，不过覆盖了put方法执行的newNode和newTreeNode和afterNodeAccess的方法。

#### newNode源码

```java
//新增节点，并追加链表尾部
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}
```

```java

private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
  LinkedHashMap.Entry<K,V> last = tail;
  //新增的节点等于位节点
    tail = p;
  //last为空说明整个链表为null 首尾节点相等
    if (last == null)
        head = p;
    else {
      //将上一节点赋值当前节点
        p.before = last;
      //新增节点加入当前节点尾部
        last.after = p;
    }
}
```

LinkedHashMap通过上面的方式 新增头节点尾结点 给节点新增before和after属性，每次新增时，都能把新节点增加到尾部节点

### 按照顺序访问

LinkedHashMap只提供单向访问，不像LinkedList的双向访问，正因为单向的，才能达到顺序访问

访问是通过迭代器进行查找访问的 迭代器初始化的时候，默认从头部节点访问 在迭代过程中，不断访问当前节点的after节点即可。

`// 初始化时，默认从头节点开始访问`
`LinkedHashIterator() {`
    `// 头节点作为第一个访问的节点`
    `next = head;`
    `expectedModCount = modCount;`
    `current = null;`
`}`

`final LinkedHashMap.Entry<K,V> nextNode() {`
    `LinkedHashMap.Entry<K,V> e = next;`
    `if (modCount != expectedModCount)// 校验`
        `throw new ConcurrentModificationException();`
    `if (e == null)`
        `throw new NoSuchElementException();`
    `current = e;`
    `next = e.after; // 通过链表的 after 结构，找到下一个迭代的节点`
    `return e;`
`}`

### 最少访问删除LRU（Least recently used 最近最少使用）

经常访问的元素会被追到队尾，这样不经常访问的数据就自然往前队头

```java
public static void testAccessOrder() {
    // 新建 LinkedHashMap
    LinkedHashMap<Integer, Integer> map = new LinkedHashMap<Integer, Integer>(4,0.75f,true) {
        {
            put(10, 10);
            put(9, 9);
            put(20, 20);
            put(1, 1);
        }

        @Override
        // 覆写了删除策略的方法，我们设定当节点个数大于 3 时，就开始删除头节点
        protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
            return size() > 3;
        }
    };
}
```

打印结果:

初始化：{9:9,20:20,1:1}
map.get(9)：{20:20,1:1,9:9}
map.get(20)：{1:1,9:9,20:20}

当调用map.get(9)方法时，元素9移动到尾部，调用map.get(20)方法时，元素20被移动到尾部。