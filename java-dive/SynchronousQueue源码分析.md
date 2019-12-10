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
		//
    SNode s = null; // constructed/reused as needed
  //e为空 说明 take 方法 不为空则是 put 方法
    int mode = (e == null) ? REQUEST : DATA;
	/// 自旋
    for (;;) {
      //拿出节点，
      //1.头节点为空，说明队列中没有有数据
      //2. 头节点不为空，并且 take 类型的， 说明 头节点线程正在拿数据
      //3. 头节点不为空 并且 put 类型的， 说明 头节点线程正在放数据
        SNode h = head;
      //栈头为空 说明队列中没有数据
      //栈头不为空，并且栈头类型和本次操作一样 那么就把put操作放在该栈头的 前面即可
        if (h == null || h.mode == mode) {  // empty or same-mode
          //设置的超间，并且 e 进栈或者出栈要超时了
          //就会丢弃本次操作， 返回null 
          //如果栈头此时被取消了 丢弃栈头 取下一个节点继续消费
            if (timed && nanos <= 0) {      // can't wait
              //栈头被取消 
              if (h != null && h.isCancelled())
                //丢弃栈头
                    casHead(h, h.next);     // pop cancelled node
                else
                    return null;
              //
            } else if (casHead(h, s = snode(s, e, h, mode))) {
                SNode m = awaitFulfill(s, timed, nanos);
                if (m == s) {               // wait was cancelled
                  //第一个元素作为栈头 
                  clean(s);
                  //返回null
                    return null;
                }
              //把新数据作为栈头
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     // help s's fulfiller
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
          //栈头正在等待其他线程 put或者 take
          //比如栈头正在阻塞 
        } else if (!isFulfilling(h.mode)) { // try to fulfill
            //栈头被取消 把下一个元素当做栈头
              if (h.isCancelled())            // already cancelled
                casHead(h, h.next);         // pop and retry
          //
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                for (;;) { // loop until matched or waiters disappear
                  //m 就是栈头  
                  SNode m = s.next;       // m is s's match
                    if (m == null) {        // all waiters are gone
                      //栈头为空 返回null
                      casHead(s, null);   // pop fulfill node
                        s = null;           // use new node next time
                        break;              // restart main loop
                    }
                    SNode mn = m.next;
                  	//tryMatch 非常重要的方法，2中作用
                  	//1唤醒被阻塞的栈头 就能从m.match中得到本次操作 s
                  	//其中 s.item 记录着本次的操作节点 本次的操作记录
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

这个方法相对比较复杂，但是从源码上可以看出来一下几点

1. 判断 put take 的方法

2. 判断栈头数据是否为空 如果为空或者栈头的操作和本次操作一致

3. 判断作为无设置超时时间 如果设置了并且已经超时 返回 null 

4. 如果栈头为空 把当前操作设置栈头或者栈头不为 null 但栈头的操作和本次操作相同

   也把当前操作设置为栈头，并且看其他线程是否满足自己，不能满足则阻塞自己 

5. 如果栈头已经被阻塞 需要别人唤醒 判当前操作能否唤醒栈头
6. 把自己当做节点赋值到栈头的 match 属性上并且唤醒栈头节点
7. 栈头被唤醒后，拿着 match 属性就把自己唤醒的节点的信息 返回

在上述方法内有个至为重要的方法 **awaitFulfill** 节点阻塞 实现原理如下

```java
SNode awaitFulfill(SNode s, boolean timed, long nanos) {
    /*
     * When a node/thread is about to block, it sets its waiter
     * field and then rechecks state at least one more time
     * before actually parking, thus covering race vs
     * fulfiller noticing that waiter is non-null so should be
     * woken.
     *
     * When invoked by nodes that appear at the point of call
     * to be at the head of the stack, calls to park are
     * preceded by spins to avoid blocking when producers and
     * consumers are arriving very close in time.  This can
     * happen enough to bother only on multiprocessors.
     *
     * The order of checks for returning out of main loop
     * reflects fact that interrupts have precedence over
     * normal returns, which have precedence over
     * timeouts. (So, on timeout, one last check for match is
     * done before giving up.) Except that calls from untimed
     * SynchronousQueue.{poll/offer} don't check interrupts
     * and don't wait at all, so are trapped in transfer
     * method rather than calling awaitFulfill.
     */
  	//死亡时间 当前时间 + 超时时间 否则就是0
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    Thread w = Thread.currentThread();
  //自旋次数 如果设置超时时间会自旋32次 否则自旋512次
  //比如这次操作是  take 操作 自旋次数后 仍然没有其他线程 put 数据进去
  //就会被阻塞 有超时时间的 会阻塞固定时间，否则一直阻塞
    int spins = (shouldSpin(s) ?
                 (timed ? maxTimedSpins : maxUntimedSpins) : 0);
    for (;;) {
      //当前线程有无被打断 
        if (w.isInterrupted())
            s.tryCancel();
        SNode m = s.match;
        if (m != null)
            return m;
        if (timed) {
            nanos = deadline - System.nanoTime();
          //超时了取消当前线程等待
            if (nanos <= 0L) {
                s.tryCancel();
                continue;
            }
        }
      //自旋次数减1
        if (spins > 0)
            spins = shouldSpin(s) ? (spins-1) : 0;
      //把当前设置 waiter 通过线程来完成阻塞和唤醒
        else if (s.waiter == null)
            s.waiter = w; // establish waiter so can park next iter
        else if (!timed)
          //通过 park 进行阻塞
            LockSupport.park(this);
        else if (nanos > spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanos);
    }
}
```

**从阻塞方法上看到 其实所谓的阻塞并不会一上来就被阻塞， 而是通过自旋一定次数 仍然没有满足要求 才会真正的被阻塞**



## 公平的队列

```java
/** 队列头 */
transient volatile QNode head;
/** 队列尾 */
transient volatile QNode tail;

// 队列的元素
static final class QNode {
    // 当前元素的下一个元素
    volatile QNode next;         
    // 当前元素的值，如果当前元素被阻塞住了，等其他线程来唤醒自己时，其他线程
    // 会把自己 set 到 item 里面
    volatile Object item;         // CAS'ed to or from null
    // 可以阻塞住的当前线程
    volatile Thread waiter;       // to control park/unpark
    // true 是 put，false 是 take
    final boolean isData;
} 
```

公平的队列主要使用 TransferQueue 内部类的 transfer 方法实现的，源码如下

```java
E transfer(E e, boolean timed, long nanos) {

    QNode s = null; // constructed/reused as needed
    // true 是 put，false 是 get
    boolean isData = (e != null);

    for (;;) {
        // 队列头和尾的临时变量,队列是空的时候，t=h
        QNode t = tail;
        QNode h = head;
        // tail 和 head 没有初始化时，无限循环
        // 虽然这种 continue 非常耗cpu，但感觉不会碰到这种情况
        // 因为 tail 和 head 在 TransferQueue 初始化的时候，就已经被赋值空节点了
        if (t == null || h == null)
            continue;
        // 首尾节点相同，说明是空队列
        // 或者尾节点的操作和当前节点操作一致
        if (h == t || t.isData == isData) {
            QNode tn = t.next;
            // 当 t 不是 tail 时，说明 tail 已经被修改过了
            // 因为 tail 没有被修改的情况下，t 和 tail 必然相等
            // 因为前面刚刚执行赋值操作： t = tail
            if (t != tail)
                continue;
            // 队尾后面的值还不为空，t 还不是队尾，直接把 tn 赋值给 t，这是一步加强校验。
            if (tn != null) {
                advanceTail(t, tn);
                continue;
            }
            //超时直接返回 null
            if (timed && nanos <= 0)        // can't wait
                return null;
            //构造node节点
            if (s == null)
                s = new QNode(e, isData);
            //如果把 e 放到队尾失败，继续递归放进去
            if (!t.casNext(null, s))        // failed to link in
                continue;

            advanceTail(t, s);              // swing tail and wait
            // 阻塞住自己
            Object x = awaitFulfill(s, e, timed, nanos);
            if (x == s) {                   // wait was cancelled
                clean(t, s);
                return null;
            }

            if (!s.isOffList()) {           // not already unlinked
                advanceHead(t, s);          // unlink if head
                if (x != null)              // and forget fields
                    s.item = s;
                s.waiter = null;
            }
            return (x != null) ? (E)x : e;
        // 队列不为空，并且当前操作和队尾不一致
        // 也就是说当前操作是队尾是对应的操作
        // 比如说队尾是因为 take 被阻塞的，那么当前操作必然是 put
        } else {                            // complementary-mode
            // 如果是第一次执行，此处的 m 代表就是 tail
            // 也就是这行代码体现出队列的公平，每次操作时，从头开始按照顺序进行操作
            QNode m = h.next;               // node to fulfill
            if (t != tail || m == null || h != head)
                continue;                   // inconsistent read

            Object x = m.item;
            if (isData == (x != null) ||    // m already fulfilled
                x == m ||                   // m cancelled
                // m 代表栈头
                // 这里把当前的操作值赋值给阻塞住的 m 的 item 属性
                // 这样 m 被释放时，就可得到此次操作的值
                !m.casItem(x, e)) {         // lost CAS
                advanceHead(h, m);          // dequeue and retry
                continue;
            }
            // 当前操作放到队头
            advanceHead(h, m);              // successfully fulfilled
            // 释放队头阻塞节点
            LockSupport.unpark(m.waiter);
            return (x != null) ? (E)x : e;
        }
    }
}
```

源码比较复杂，线程被阻塞后 当前线程是如何把自己的数据传给阻塞线程的 假设线程1 往队列中 take 数据 被阻塞了 变成阻塞线程A 然后线程2开始往队列里 put B

1. 线程1从队列中拿到数据 发现队列中没有数据 于是被阻塞 成为A
2. 线程2往队尾 put 放数据 会从队尾往前找到第一个阻塞的节点 假设此时能找到的就是节点A然后线程B将 put数据放到节点A的item属性里面 并唤醒线程1
3. 线程1被唤醒就从 A.item 里拿到线程2 put 的数据了 线程1成功返回

## 总结

SynchrononsQueue是非常难的 源码也很难理解~~~~