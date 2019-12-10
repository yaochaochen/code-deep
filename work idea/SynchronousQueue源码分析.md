# SynchronousQueue源码分析



## 引导语

SynchronousQueue是一个比较独特的队列，其本身没有容量大小，比如我放一个书记到队列中，我是不能够立马返回的，我必须等待别人把我放进去的数据消费掉，才能够返回

SynchronousQueue在消息队列大量使用

## 整体架构

SynchronousQueue设计比较抽象，在内部抽象出两种算法实现，一种是先入先出的队列，一种是后入先出的堆栈，两种算法被两个内部类实现，而直接对外put take方法的实现就比较简单，直接调用 transfer 方法进行实现

### 类注释

```java
/**
 * A {@linkplain BlockingQueue blocking queue} in which each insert
 * operation must wait for a corresponding remove operation by another
 * thread, and vice versa.  A synchronous queue does not have any
 * internal capacity, not even a capacity of one.  You cannot
 * {@code peek} at a synchronous queue because an element is only
 * present when you try to remove it; you cannot insert an element
 * (using any method) unless another thread is trying to remove it;
 * you cannot iterate as there is nothing to iterate.  The
 * <em>head</em> of the queue is the element that the first queued
 * inserting thread is trying to add to the queue; if there is no such
 * queued thread then no element is available for removal and
 * {@code poll()} will return {@code null}.  For purposes of other
 * {@code Collection} methods (for example {@code contains}), a
 * {@code SynchronousQueue} acts as an empty collection.  This queue
 * does not permit {@code null} elements.
 *
 * <p>Synchronous queues are similar to rendezvous channels used in
 * CSP and Ada. They are well suited for handoff designs, in which an
 * object running in one thread must sync up with an object running
 * in another thread in order to hand it some information, event, or
 * task.
 *
 * <p>This class supports an optional fairness policy for ordering
 * waiting producer and consumer threads.  By default, this ordering
 * is not guaranteed. However, a queue constructed with fairness set
 * to {@code true} grants threads access in FIFO order.
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
 * @author Doug Lea and Bill Scherer and Michael Scott
 * @param <E> the type of elements held in this collection
 */
```

源码注释可以看出来

- 队列不存储数据，所有没有大小，而无法迭代
- 插入操作的必须等待另一个线程完成对应数据的删除操作
- 队列由于两种数据结构组成，分别是后入先出的堆栈和先进先出的队列，堆栈是不公平的，队列是公平

看到类注释比较好奇的是 如何实现另外线程操作呢？怎么在堆栈上实现后出先入呢？

### 类图

SynchronousQueue 整体类图和 LinkedBlockingQueue 相似 都实现了 BlockingQueue 接口 但是因为不保存数据结结构 一些方法是没有实现的比如 isEmpty、 size、 contains、 remove、 forEach 等方法

 

```java
/**
 * Does nothing.
 * A {@code SynchronousQueue} has no internal capacity.
 */
public void clear() {
}

/**
 * Always returns {@code false}.
 * A {@code SynchronousQueue} has no internal capacity.
 *
 * @param o the element
 * @return {@code false}
 */
public boolean contains(Object o) {
    return false;
}

/**
 * Always returns {@code false}.
 * A {@code SynchronousQueue} has no internal capacity.
 *
 * @param o the element to remove
 * @return {@code false}
 */
public boolean remove(Object o) {
    return false;
}

/**
 * Returns {@code false} unless the given collection is empty.
 * A {@code SynchronousQueue} has no internal capacity.
 *
 * @param c the collection
 * @return {@code false} unless given collection is empty
 */
public boolean containsAll(Collection<?> c) {
    return c.isEmpty();
}

/**
 * Always returns {@code false}.
 * A {@code SynchronousQueue} has no internal capacity.
 *
 * @param c the collection
 * @return {@code false}
 */
public boolean removeAll(Collection<?> c) {
    return false;
}

/**
 * Always returns {@code false}.
 * A {@code SynchronousQueue} has no internal capacity.
 *
 * @param c the collection
 * @return {@code false}
 */
public boolean retainAll(Collection<?> c) {
    return false;
}

/**
 * Always returns {@code null}.
 * A {@code SynchronousQueue} does not return elements
 * unless actively waited on.
 *
 * @return {@code null}
 */
public E peek() {
    return null;
}

/**
 * Returns an empty iterator in which {@code hasNext} always returns
 * {@code false}.
 *
 * @return an empty iterator
 */
public Iterator<E> iterator() {
    return Collections.emptyIterator();
}

/**
 * Returns an empty spliterator in which calls to
 * {@link java.util.Spliterator#trySplit()} always return {@code null}.
```

以上方法都是空实现或者默认实现

### 结构细节

SynchronousQueue 底层结构和其他队列完全不同，有2中数据结构，队列和堆栈

```java
//抽象类，堆栈和队列共同实现
abstract static class Transferer<E> {
    /**
     * Performs a put or take.
     *
     * @param e if non-null, the item to be handed to a consumer;
     *          if null, requests that transfer return an item
     *          offered by producer.
     * @param timed if this operation should timeout
     * @param nanos the timeout, in nanoseconds
     * @return if non-null, the item provided or received; if null,
     *         the operation failed due to timeout or interrupt --
     *         the caller can distinguish which of these occurred
     *         by checking Thread.interrupted.
     */
  //e为空直接返回特殊值， 不为空传递给消费者
  //timed为 true 说明超时时间
    abstract E transfer(E e, boolean timed, long nanos);
}
```

```java
//队列 先入先出 公平的
/** Dual Queue */
static final class TransferQueue<E> extends Transferer<E> 
```

```java
///堆栈 后入先出 非公平的
static final class TransferStack<E> extends Transferer<E> {
```

```java
/**
 * Creates a {@code SynchronousQueue} with the specified fairness policy.
 *
 * @param fair if true, waiting threads contend in FIFO order for
 *        access; otherwise the order is unspecified.
 */
//无参构造 默认是非公平的
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
}
```

从源码上我们可以得到几点：

1. 堆栈和队列都有一个共同接口，叫做 Transferer 该接口有个方法 transferer 该方法承担了 take 和 put 的双重功能
2. 在我们初始化的时候，可以选择使用堆栈和队列 如果不选择，默认就是堆栈，堆栈效率会高。



## 堆栈

### 堆栈结构

| put放在堆栈头 take 从堆栈头拿 |
| :---------------------------: |
|         栈元素SNode1          |
|         栈元素SNode2          |
|         栈元素SNode3          |

从上我们可以看出来，我们有一个大的堆栈池，池的开口叫做堆栈头，put 的时候 就往堆栈池中放数据 take 的时候 就从堆栈池中拿数据 两者都在堆栈头上操作数据，越靠近堆栈头 数据越新 所以每次 take 的时候 都是最新的数据 这就是后入先出 也是非公平的

所以我们关注下SNode的具体实现

```java
static final class SNode {
    volatile SNode next;        // next node in stack
    volatile SNode match;       // the node matched to this
    volatile Thread waiter;     // to control park/unpark
    Object item;                // data; or null for REQUESTs
    int mode;
    // Note: item and mode fields don't need to be volatile
    // since they are always written before, and read after,
    // other volatile/atomic operations.

    SNode(Object item) {
        this.item = item;
    }

    boolean casNext(SNode cmp, SNode val) {
        return cmp == next &&
            UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }
```

### 入栈和出栈

入栈是使用 put 等方法，把数据放入堆栈池中，出栈指的使用 take 等方法 把数据从堆栈池拿出来

```java
E transfer(E e, boolean timed, long nanos) {
    /*
     * Basic algorithm is to loop trying one of three actions:
     *
     * 1. If apparently empty or already containing nodes of same
     *    mode, try to push node on stack and wait for a match,
     *    returning it, or null if cancelled.
     *
     * 2. If apparently containing node of complementary mode,
     *    try to push a fulfilling node on to stack, match
     *    with corresponding waiting node, pop both from
     *    stack, and return matched item. The matching or
     *    unlinking might not actually be necessary because of
     *    other threads performing action 3:
     *
     * 3. If top of stack already holds another fulfilling node,
     *    help it out by doing its match and/or pop
     *    operations, and then continue. The code for helping
     *    is essentially the same as for fulfilling, except
     *    that it doesn't return the item.
     */

    SNode s = null; // constructed/reused as needed
    int mode = (e == null) ? REQUEST : DATA;

    for (;;) {
        SNode h = head;
        if (h == null || h.mode == mode) {  // empty or same-mode
            if (timed && nanos <= 0) {      // can't wait
                if (h != null && h.isCancelled())
                    casHead(h, h.next);     // pop cancelled node
                else
                    return null;
            } else if (casHead(h, s = snode(s, e, h, mode))) {
                SNode m = awaitFulfill(s, timed, nanos);
                if (m == s) {               // wait was cancelled
                    clean(s);
                    return null;
                }
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     // help s's fulfiller
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
        } else if (!isFulfilling(h.mode)) { // try to fulfill
            if (h.isCancelled())            // already cancelled
                casHead(h, h.next);         // pop and retry
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                for (;;) { // loop until matched or waiters disappear
                    SNode m = s.next;       // m is s's match
                    if (m == null) {        // all waiters are gone
                        casHead(s, null);   // pop fulfill node
                        s = null;           // use new node next time
                        break;              // restart main loop
                    }
                    SNode mn = m.next;
                    if (m.tryMatch(s)) {
                        casHead(s, mn);     // pop both s and m
                        return (E) ((mode == REQUEST) ? m.item : s.item);
                    } else                  // lost match
                        s.casNext(m, mn);   // help unlink
                }
            }
        } else {                            // help a fulfiller
            SNode m = h.next;               // m is h's match
            if (m == null)                  // waiter is gone
                casHead(h, null);           // pop fulfilling node
            else {
                SNode mn = m.next;
                if (m.tryMatch(h))          // help match
                    casHead(h, mn);         // pop both h and m
                else                        // lost match
                    h.casNext(m, mn);       // help unlink
            }
        }
    }
}
```

