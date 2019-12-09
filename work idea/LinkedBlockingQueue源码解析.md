# LinkedBlockingQueue源码解析

## 前言

说起队列，大家基本上都会说都没有用过，应该是不重要的API，如果这样想，那就打错特错了，我们平时使用的线程池，读写锁 消息队列等技术框架 底层原理都是队列。所以不要轻视队列，队列很多高级的API。

## 类图

1. AbstractQueue->AbstractCollection->Collection->Iterable 主要想复制Collection和迭代的操作。
2. BlockingQueue->Queue->Collection BlockingQueue和Queue是新增的接口

Queue是基础接口，几乎所有的队列的实现这个接口，该接口定义了三个大类操作

新增操作:

1. add队列满的时候抛出错误
2. offer队列满时会返回false

查看并删除操作：

1. element队列为空的时候抛错
2. peek队列为空的时候返回null

一共6种方法，除了以上分类方法，也可以分成两类:

1. 遇到队列满或者空的时候，抛异常如 add remove element
2. 遇到队列满或者空的时候，返回特殊值 offer poll peek

|                   | 抛错    | 特殊值         | 一直阻塞 | 阻塞一段时间           |
| ----------------- | ------- | -------------- | -------- | ---------------------- |
| 新增操作-队列满   | add     | offer返回false | put      | offer过超时时间 false  |
| 查看并删除-队列空 | remove  | poll返回null   | Take     | poll过超时时间返回null |
| 只查看不删除      | element | peek返回null   | 暂无     | 暂无                   |

## 类注释

```java
/**
 * An optionally-bounded {@linkplain BlockingQueue blocking queue} based on
 * linked nodes.
 * This queue orders elements FIFO (first-in-first-out).
 * The <em>head</em> of the queue is that element that has been on the
 * queue the longest time.
 * The <em>tail</em> of the queue is that element that has been on the
 * queue the shortest time. New elements
 * are inserted at the tail of the queue, and the queue retrieval
 * operations obtain elements at the head of the queue.
 * Linked queues typically have higher throughput than array-based queues but
 * less predictable performance in most concurrent applications.
 *
 * <p>The optional capacity bound constructor argument serves as a
 * way to prevent excessive queue expansion. The capacity, if unspecified,
 * is equal to {@link Integer#MAX_VALUE}.  Linked nodes are
 * dynamically created upon each insertion unless this would bring the
 * queue above capacity.
 *
 * <p>This class and its iterator implement all of the
 * <em>optional</em> methods of the {@link Collection} and {@link
 * Iterator} interfaces.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <E> the type of elements held in this collection
 */
```

1. 基于链表的阻塞队列，其底层的数据结构是链表
2. 链表维护FIFO模式，新元素放在队尾，获取元素从队头拿
3. 链表大小在初始化的时候可以设置，模式Integer的最大值
4. 可以使用Collection和Iterator两个接口的所有操作，因为实现它们的接口

## 内部实现

LinkedBlockingQueue内部结构 分成三部分 链表+锁+迭代器 

```java
/**
 * Linked list node class
 */
//链表结构的元素
static class Node<E> {
    E item;

    /**
     * One of:
     * - the real successor Node
     * - this Node, meaning the successor is head.next
     * - null, meaning there is no successor (this is the last node)
     */
    Node<E> next;

    Node(E x) { item = x; }
}
//链表容量 默认值 Integer.MAX_VALUE
/** The capacity bound, or Integer.MAX_VALUE if none */
private final int capacity;

//链表已有元素， 所有线程是安全的
/** Current number of elements */
private final AtomicInteger count = new AtomicInteger();

/**
 * Head of linked list.
 * Invariant: head.item == null
 */
//链表头部
transient Node<E> head;

/**
 * Tail of linked list.
 * Invariant: last.next == null
 */
//连表尾部
private transient Node<E> last;

//锁 可重入锁
/** Lock held by take, poll, etc */
private final ReentrantLock takeLock = new ReentrantLock();

// take时的锁 ASQ同步机制
/** Wait queue for waiting takes */
private final Condition notEmpty = takeLock.newCondition();

//put时候锁 可以实现take和put锁同步
/** Lock held by put, offer, etc */
private final ReentrantLock putLock = new ReentrantLock();
 //Put条件锁
/** Wait queue for waiting puts */
private final Condition notFull = putLock.newCondition();
```

1. 链表的作用为了保存当前节点，节点中的数据可以是任何东西，是一个泛型也可以
2. 锁有take和put 为了保证队列操作时是线程安全 设计两种锁，是为了take和put两种可以同时操作 互不影响

## 初始化

初始化有三种方式:

1. 指定链表容量大小
2. 不指定链表大小，默认是Integer的最大值
3. 对已有集合数据进行初始化

```java
/**
 * Creates a {@code LinkedBlockingQueue} with a capacity of
 * {@link Integer#MAX_VALUE}.
 */
//不指定容量，默认Integer的最大值
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}

/**
 * Creates a {@code LinkedBlockingQueue} with the given (fixed) capacity.
 *
 * @param capacity the capacity of this queue
 * @throws IllegalArgumentException if {@code capacity} is not greater
 *         than zero
 */
//指定大小，如果指定的大小为0会抛错
public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
  //收尾链表相等
    last = head = new Node<E>(null);
}

/**
 * Creates a {@code LinkedBlockingQueue} with a capacity of
 * {@link Integer#MAX_VALUE}, initially containing the elements of the
 * given collection,
 * added in traversal order of the collection's iterator.
 *
 * @param c the collection of elements to initially contain
 * @throws NullPointerException if the specified collection or any
 *         of its elements are null
 */
//已有集合初始化
public LinkedBlockingQueue(Collection<? extends E> c) {
    this(Integer.MAX_VALUE);
    final ReentrantLock putLock = this.putLock;
    putLock.lock(); // Never contended, but necessary for visibility
    try {
        int n = 0;
        for (E e : c) {
            if (e == null)
              //集合为空
                throw new NullPointerException();
            if (n == capacity)
              //当前集合的大小大于MAX_VALUE会抛错
                throw new IllegalStateException("Queue full");
            enqueue(new Node<E>(e));
            ++n;
        }
        count.set(n);
    } finally {
        putLock.unlock();
    }
}
```

### 说明补充:

- 初始化时，容量的大小是不会影响性能的，只影响后面的使用，因为指定太小，容易导致没有放入的队列就会有已满的错误
- 在集合数据初始化，源码给看一个不优雅的示范，我们不反对每次for循环的时候检查当前链表的大小是否超过容量，但我们希望在for循环开始之前做检查。

## 阻塞新增

新增有很多方法，不如add put offer 三者区别上文有说，我们拿put方法为例，put方法在碰到队列满的时候，会一直阻塞下去，知道队列不满时，才会被自己唤醒

```java
/**
 * Inserts the specified element at the tail of this queue, waiting if
 * necessary for space to become available.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
//把e新增的队列放在尾部
//如果新增时有空间则直接新增成功，否则当前线程陷入等待
public void put(E e) throws InterruptedException {
  //e为空抛错
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
  //约定数，当为-1新增失败
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
  //中断锁
    putLock.lockInterruptibly();
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
      //队列满了
      //当前线程阻塞，等待其他线程唤醒
        while (count.get() == capacity) {
            notFull.await();
        }
      //队列不满 新增
        enqueue(node);
        c = count.getAndIncrement();
      //小于链表容量，说明队列未满
      //唤起线程
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
      //释放线程锁
        putLock.unlock();
    }
  //有一个元素
    if (c == 0)
        signalNotEmpty();
}
```

### 说明补充

1. 往队列里新增时，第一步就是加锁，新增数据是安全的
2. 往队列新增数据，简单的追加到尾部
3. 新增时如果队列满了，当前线程是会被阻塞的，阻塞的底层是使用锁的能力
4. 新增数据成功后，在适当时机 会唤醒put的等待线程，或者take的等待线程 这样保证一旦满足put或者take条件时，立马就能唤起阻塞线程，继续运行

offer会阻塞一段时间，扔未成功，就直接返回false 

```java
 public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        if (e == null) throw new NullPointerException();
        long nanos = unit.toNanos(timeout);
        int c = -1;
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            while (count.get() == capacity) {
              //等待时间小于零会等待线程
                if (nanos <= 0)
                    return false;
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(new Node<E>(e));
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
        return true;
    }
```

## 阻塞删除

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 

```java
public E take() throws InterruptedException {
    E x;
 		//删除失败时
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
      //空队列时，等待线程，其他线程唤醒
        while (count.get() == 0) {
            notEmpty.await();
        }
      //非空队列，从队列头部哪一个
        x = dequeue();
      //减一
        c = count.getAndDecrement();
      //如果队列有值，从take的等待线程中唤醒一个
      //唤醒之前被阻塞的队列
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

## 总结

队列本身是一个阻塞工具，我们可以把这个工具应用到阻塞场景里，比如队列应用到线程池上，当线程跑满时，我们吧新的请求放到阻塞队列等待

消息队列，当前消费者处理能力有限时，我们吧消息放入队列等待，让消费者慢慢消费