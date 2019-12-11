# DelayQueue源码分析

之前我们说的阻塞队列，都是资源足够时立马执行。这次介绍的DelayQueue是一个延时队列，并且可以设置延迟多久之后执行，比如设置5秒后再执行，在一些延迟执行场景被大量使用，不如延迟对账。

## 整体设计

DelayQueue延迟队列底层使用是锁的能力，比如说要当前时间往后延迟5秒后执行，那么当前线程 就会沉睡5秒，等5秒后线程被唤醒时，如果能获得资源的话，线程即可立马执行，原理上似乎很简单，但是内部实现比SynchronousQueue复杂的多，也有很多难点，比如运行资源不足时，多线程同时被唤醒时，如何排队等待，何时阻塞，何时开启等待.....

### 类注释

```java
/**
 * An unbounded {@linkplain BlockingQueue blocking queue} of
 * {@code Delayed} elements, in which an element can only be taken
 * when its delay has expired.  The <em>head</em> of the queue is that
 * {@code Delayed} element whose delay expired furthest in the
 * past.  If no delay has expired there is no head and {@code poll}
 * will return {@code null}. Expiration occurs when an element's
 * {@code getDelay(TimeUnit.NANOSECONDS)} method returns a value less
 * than or equal to zero.  Even though unexpired elements cannot be
 * removed using {@code take} or {@code poll}, they are otherwise
 * treated as normal elements. For example, the {@code size} method
 * returns the count of both expired and unexpired elements.
 * This queue does not permit null elements.
 *
 * <p>This class and its iterator implement all of the
 * <em>optional</em> methods of the {@link Collection} and {@link
 * Iterator} interfaces.  The Iterator provided in method {@link
 * #iterator()} is <em>not</em> guaranteed to traverse the elements of
 * the DelayQueue in any particular order.
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

从上面类注释可以看出来三点：

1. 队列中元素将在过期时被执行，越靠近队头，越早过期；
2. 未过期的元素不能够被 take
3. 不允许为空

这三个概念，其实就是三个问题，接下我们看看如何实现

------

### 类图

DelayQueue 的类图和之前的队列一样，关键是 DelayQueue 类上有泛型的

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E> {
```

从泛型中可以看出来，DelayQueue 中的元素必须是 Delayed 的子类，Delayed 是表达延迟能力的关键接口，

其继承 Comparable 接口，并且定义了还剩多久过期的方法。

```java
public interface Delayed extends Comparable<Delayed> {
   long getDelay(TimeUnit unit);
}
```

也就是说，DelayQueue队列中的元素必须是实现Delayed的接口和Comparable接口，并且覆盖getDelay 的方法和compareTo方法才行。不然编译时，编译器就会提醒元素必须强制实现Delayed接口。

除此之外DelayQueue 还大量使用 PriorityQueue 队列的大量功能，这个和SynchronousQueue队列很像

```java
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
      //q就是PriorityQueue
        E first = q.peek();
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
            return q.poll();
    } finally {
        lock.unlock();
    }
}
```

PriorityQueue 也是优先级队列，在此处的作用就是可以根据过期时间做优先级排序，让先过期的可以执行

这里用的复用的思想还很重要，我们在源码中经常会遇到这种思想，比如说LinkedHashMap 复用HashMap的能力。Set 复用 Map 的能力，还有此处 DelayQueue 复用 PriorityQueue 的能力。总结一下几点：



1. 需要把能遇见可复用的功能尽量抽象，并且开发出扩展的地方，比如说 HashMap在操作数组的方法中，都给 LinkedHashMap 开放出来很多 after 开头的方法，便于 LinkedHashMap进行排序、删除等等；
2. 采用组合或者继承两种手段进行复用，比如 LinkedHashMap采用继承，Set 和 DelayQueue 采用的组合，组合的意思就是把可用的类给依赖进来。

## 演示

```java
public class DelayQueueDemo {
	// 队列消息的生产者
  static class Product implements Runnable {
    private final BlockingQueue queue;
    public Product(BlockingQueue queue) {
      this.queue = queue;
    }
    
    @Override
    public void run() {
      try {
        log.info("begin put");
        long beginTime = System.currentTimeMillis();
        queue.put(new DelayedDTO(System.currentTimeMillis() + 2000L,beginTime));//延迟 2 秒执行
        queue.put(new DelayedDTO(System.currentTimeMillis() + 5000L,beginTime));//延迟 5 秒执行
        queue.put(new DelayedDTO(System.currentTimeMillis() + 1000L * 10,beginTime));//延迟 10 秒执行
        log.info("end put");
      } catch (InterruptedException e) {
        log.error("" + e);
      }
    }
  }
	// 队列的消费者
  static class Consumer implements Runnable {
    private final BlockingQueue queue;
    public Consumer(BlockingQueue queue) {
      this.queue = queue;
    }

    @Override
    public void run() {
      try {
        log.info("Consumer begin");
        ((DelayedDTO) queue.take()).run();
        ((DelayedDTO) queue.take()).run();
        ((DelayedDTO) queue.take()).run();
        log.info("Consumer end");
      } catch (InterruptedException e) {
        log.error("" + e);
      }
    }
  }

  @Data
  // 队列元素，实现了 Delayed 接口
  static class DelayedDTO implements Delayed {
    Long s;
    Long beginTime;
    public DelayedDTO(Long s,Long beginTime) {
      this.s = s;
      this.beginTime =beginTime;
    }

    @Override
    public long getDelay(TimeUnit unit) {
      return unit.convert(s - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    @Override
    public int compareTo(Delayed o) {
      return (int) (this.getDelay(TimeUnit.MILLISECONDS) - o.getDelay(TimeUnit.MILLISECONDS));
    }

    public void run(){
      log.info("现在已经过了{}秒钟",(System.currentTimeMillis() - beginTime)/1000);
    }
  }
	// demo 运行入口
  public static void main(String[] args) throws InterruptedException {
    BlockingQueue q = new DelayQueue();
    DelayQueueDemo.Product p = new DelayQueueDemo.Product(q);
    DelayQueueDemo.Consumer c = new DelayQueueDemo.Consumer(q);
    new Thread(c).start();
    new Thread(p).start();
  }
}
打印出来的结果如下：
06:57:50.544 [Thread-0] Consumer begin
06:57:50.544 [Thread-1] begin put
06:57:50.551 [Thread-1] end put
06:57:52.554 [Thread-0] 延迟了2秒钟才执行
06:57:55.555 [Thread-0] 延迟了5秒钟才执行
06:58:00.555 [Thread-0] 延迟了10秒钟才执行
06:58:00.556 [Thread-0] Consumer end
```

1. 新建队列元素，如DelayedDTO,必须实现Delayed接口，我们在getDelay方法中实现了现在离过期时间还剩多久的方法。
2. 自定义队列元素的生成者，和消费者 对应着代码Product 和 Consumer 
3. 对生产者和消费者进行初始化话管理，对应 main 方法

## 放数据

以put的为例，put调用是offer方法，offer方法源码

```java
/**
 * Inserts the specified element into this delay queue.
 *
 * @param e the element to add
 * @return {@code true}
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
  //上锁
    lock.lock();
    try {
      //PriorityQueue的扩容，排序能力
        q.offer(e);
      //如果恰好放在元素队列头
      //立马唤醒 take 的阻塞线程，执行take 操作
      //如果元素需要延迟执行的话，可以使其更快沉睡计时
        if (q.peek() == e) {
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```

接下来我们看 PriorityQueue 的offer 方法

```java
/**
 * Inserts the specified element into this priority queue.
 *
 * @return {@code true} (as specified by {@link Queue#offer})
 * @throws ClassCastException if the specified element cannot be
 *         compared with elements currently in this priority queue
 *         according to the priority queue's ordering
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
  //元素为空 异常
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
  //队列实际大小大于容量时，进行扩容
  //扩容策略是，如果老扩容小于64 2倍扩容，如果大于64 1/2扩容
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
  //如果队列为空，当前元素正好处于队头
    if (i == 0)
        queue[0] = e;
    else
      //需要根据优先级进行排序
        siftUp(i, e);
    return true;
}
```

```java
//按照从小到大的顺序排序
private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
  //当前队列大小
    while (k > 0) {
      //对 k 进行减倍
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
      //如果 x 比 e 大 退出 把 x 放在 k 的位置上
        if (key.compareTo((E) e) >= 0)
            break;
      // x 比 e 小继续循环，直接找到 x 比队列中元素大的位置
        queue[k] = e;
        k = parent;
    }
    queue[k] = key;
}
```

offer主要做了三件事

1. 对新增元素进行判空
2. 对队列进行扩容，扩容策略和集合的策略很像
3. 根据元素的compareTo 方法进行排序

```java
(int) (this.getDelay(TimeUnit.MILLISECONDS) - o.getDelay(TimeUnit.MILLISECONDS));
```

## 拿数据

取数据时，如果发现有元素的过期时间到了，就能拿出来数据来，如果没有过期元素，那么线程就会一直被阻塞。

```java
for (;;) {
    // 从队头中拿数据出来
    E first = q.peek();
    // 如果为空，说明队列中，没有数据，阻塞住
    if (first == null)
        available.await();
    else {
        // 获取队头数据的过期时间
        long delay = first.getDelay(NANOSECONDS);
        // 如果过期了，直接返回队头数据
        if (delay <= 0)
            return q.poll();
        // 引用置为 null ，便于 gc，这样可以让线程等待时，回收 first 变量
        first = null;
        // leader 不为空的话，表示当前队列元素之前已经被设置过阻塞时间了
        // 直接阻塞当前线程等待。
        if (leader != null)
            available.await();
        else {
          // 之前没有设置过阻塞时间，按照一定的时间进行阻塞
            Thread thisThread = Thread.currentThread();
            leader = thisThread;
            try {
                // 进行阻塞
                available.awaitNanos(delay);
            } finally {
                if (leader == thisThread)
                    leader = null;
            }
        }
    }
}
```

