# Future、ExecutorService源码分析

------

## Callable

Callable 是一个接口， 约定了线程要做的事情，和 Runnable 一样，不过这个线程任务是有返回值的，看下接口定义

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

返回值是一个泛型，可以定义成任何类型，但在使用的过程中，都不会直接使用 Callable 而是结合FutureTask 一起使用

## FutureTask

FutureTask 可以当做是线程运行的具体任务，我们可以看到 FutureTask 实现了 RunnableFutre 接口 

```java
public class FutureTask<V> implements RunnableFuture<V> {
```

而 RunnableFutureTask 又实现了 Runnable,Future 两个接口，先看 Future， 再看 RunnableFuture 最后 FutureTask

###  Callable

 是可以返回子线程执行结果，在获得结果的结果，就需要 Future接口了。

Future 接口注释写了

```java
/**
 * Attempts to cancel execution of this task.  This attempt will
 * fail if the task has already completed, has already been cancelled,
 * or could not be cancelled for some other reason. If successful,
 * and this task has not started when {@code cancel} is called,
 * this task should never run.  If the task has already started,
 * then the {@code mayInterruptIfRunning} parameter determines
 * whether the thread executing this task should be interrupted in
 * an attempt to stop the task.
 *
 * <p>After this method returns, subsequent calls to {@link #isDone} will
 * always return {@code true}.  Subsequent calls to {@link #isCancelled}
 * will always return {@code true} if this method returned {@code true}.
 *
 * @param mayInterruptIfRunning {@code true} if the thread executing this
 * task should be interrupted; otherwise, in-progress tasks are allowed
 * to complete
 * @return {@code false} if the task could not be cancelled,
 * typically because it has already completed normally;
 * {@code true} otherwise
 */
```

1. 定义了异步计算接口，提供了计算是否完成的check、等待完成和取回多种方法
2. 如果想得到结果可以使用 get 方法，此方法会一直阻塞到异步任务计算完成
3. 取消可以使用 cancel 方法，但一旦任务计算完成，就无法取消

Future 接口定义了这些方法

```java
//如果任务已经完成，或者已经取消了 是无法再取消 的 会直接返回 true 
//如果任务还没有开始进行时，发起取消 是可以取消成功的
//如果取消时，任务已经在运行时， mayInterruptRunning 为 true 的话， 就可以打断运行中线程
  boolean cancel(boolean mayInterruptIfRunning);

   //返回线程已经被取消 true 表示你已经被取消了
	//如果线程已经运行结束了，IsCanceelled 和 isDone 返回都是 true 
    boolean isCancelled();

  //线程是否已经运行结束了
    boolean isDone();

    //等待结果返回
		//如果任务被取消，抛CancellationException 异常
		//如果等待过程中被打断了，抛出 InterruptedException
    V get() throws InterruptedException, ExecutionException;

    
		//等待结果返回，带有超时时间 超过超时时间会抛错
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```

从接口上可以看看，Future 定义了各种方法对任务进行管理，比如取消任务，得到任务 计算结果等等

### RunnableFuture

RunnableFuture 是一个接口， 定义如下:

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

RunnableFuture 接口的最大目的，是让 Future 可以对Runnable进行管理，可以取消 Runnable 是否完成等等

### 统一 Callable 和 Runnable

 新建任务有2种方式，一种是无返回值的 Runable 一种是有返回值的 Callable ，但是对Java 其他API 来说使用起来不是方便的，没有统一的接口，比如线程池提交任务时，是不是应该针对 Runnable 和 Callable 两种情况提供不同的实现思路，所以 FutureTask 出现了， FutureTask 实现了 RunnableFuture 接口，又集合了 Callable ，而已还提供两者转化的方法， 这样下来， FutureTask就统一了 Callable 和 Runnable.

#### FutureTask的类定义

```java
public class FutureTask<V> implements RunnableFuture<V> 
```

从类定义上可以看出来，FutureTask 实现了 RunnableFuture 接口，也就是说间接实现了，Runnable 接口，就是说 FutureTask 本身就是 Runnable  同时FutureTask 也是实现了 Future 也就是说，FutureTask 具备对任务进行管理功能

#### FutureTask的属性

我们一起来看 FutureTask 的重要的属性

```java
/**
 * The run state of this task, initially NEW.  The run state
 * transitions to a terminal state only in methods set,
 * setException, and cancel.  During completion, state may take on
 * transient values of COMPLETING (while outcome is being set) or
 * INTERRUPTING (only while interrupting the runner to satisfy a
 * cancel(true)). Transitions from these intermediate to final
 * states use cheaper ordered/lazy writes because values are unique
 * and cannot be further modified.
 *
 * Possible state transitions:
 * NEW -> COMPLETING -> NORMAL
 * NEW -> COMPLETING -> EXCEPTIONAL
 * NEW -> CANCELLED
 * NEW -> INTERRUPTING -> INTERRUPTED
 */
//任务状态
private volatile int state;
private static final int NEW          = 0; // 新建
private static final int COMPLETING   = 1; // 任务执行中
private static final int NORMAL       = 2; //任务执行结束
private static final int EXCEPTIONAL  = 3; //任务异常
private static final int CANCELLED    = 4; // 任务取消成功
private static final int INTERRUPTING = 5; // 任务执行中被打断
private static final int INTERRUPTED  = 6; //任务被打断成功

/**组合了 Callable The underlying callable; nulled out after running */
private Callable<V> callable;
/** 异步线程执行返回的结果 The result to return or exception to throw from get() */
private Object outcome; // non-volatile, protected by state reads/writes
/** 当前任务执行的所有线程 The thread running the callable; CASed during run() */
private volatile Thread runner;
/** 记录调用 get 方法时被等待 的线程 Treiber stack of waiting threads */
private volatile WaitNode waiters;
```

从属性上，我们明显看到 Callable 是作为 FutureTask 的属性之一，这也就让 FutureTask 具备了转化 Callable 和 Runnable 的功能。

####   FutureTask 的构造器

```java
//使用 Callable 进行初始化
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

```java
//使用 Runnable 进行初始化 并传入 result 作为返回结果
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

这2个构造器，只有一个目的，就是把入参都转化成 Callable 为什么不转成 Runnable呢？ 主要是因为   Callable 的功能比 Runnable 丰富 Callable有返回值， 而且 Runnable 没有。

我们注意到入参是 Runnable  的构造器 会使用 Executor.callable 方法 来把 Runnable 转化成Callable Runnable 和 Callable 都是接口， 两者之间 无法进行转化，所以 Java 提供了 RunnableAdapter 来进行转化

```java
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

1. 首先 RunnableAdapter 实现了 Callable 所以 RunnableAdapter 就是 Callable 
2. 其次 Runnable 是 RunnableAdapter 一个属性 在构造 RunnableAdapter 的时候会传进来，并且在 call 方法里面调用 Runnable 的 run 方法

这是一个典型的适配模式，我们把 Runnable 适配成 Callable 首先实现 Callable 的接口，接着 Callable 的 call 方法里面调用被适配对象的方法

这种构造器思想设计的很巧妙，将 Runnable 和 Callable 灵活的打通，向内和向外只提供功能更加丰富的 Callable 接口。值得我们学习

#### FutureTask 和 Future 接口 方法实现 

##### get

get 有无限阻塞和带超时时间2种方法，我们通常建议使用带超时时间的方法。

```java
/**
 * @throws CancellationException {@inheritDoc}
 */
public V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();
    int s = state;
  //如果任务已经在执行中了，并且等待一定时间后，仍然在 执行中，直接抛错
    if (s <= COMPLETING &&
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)
        throw new TimeoutException();
  //执行成功 返回执行的结果 
    return report(s);
}
```

```java
//等待任务执行完成
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
  //计算等待终止时间，如果一直等待你的话，终止 时间为 0
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
  //不排队
    boolean queued = false;
  //无限循环
    for (;;) {
      //线程被中断，删除抛错
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }
    //当前线程状态
        int s = state;
      //已被执行
        if (s > COMPLETING) {
            if (q != null)
              //当前线程置null
                q.thread = null;
            return s;
        }
      //如果正在执行，当前线程让出 CPU 重新竞争，防止 CPU 飙升
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
      //第一次执行
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}
```

get 方法虽然名字叫做 get 但做了很多 wait 的事情 当发生任务还是在进行中， 没有完成时， 就会阻塞当前进程，等待任务完成后 再返回结果集，阻塞底层使用的是， LockSupport.park 方法 使得当前 线程进入WAITING 或者 TIME_WAITING 状态。

#####  run

```java
// run 方法直接被调用 也可以开启新的线程进行调用
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
              //调用执行
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

1.  run 方法没有返回值， 通过 outcome 属性赋值 get 就能从 outcome 属性中拿到返回值
2. FutureTask 两种构造器 最终都转化成了 Callable 所以在 run 方法执行的时候，只需要执行 Callable 的 call 方法即可，在执行c.call() 代码时，如果入参在 Runnable 的话，调用路径 c.call()-> RunnableAdapter.call() -> Runable.run() 如果入参是 Callable 的话，直接调用

#####  cancel

```java
//取消任务，如果 正在运行，尝试打断
public boolean cancel(boolean mayInterruptIfRunning) {
  //如果不是 new 状态置为取消 直接返回false 
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {    // in case call to interrupt throws exception
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { 
              //// final state 设置打断
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
      //清理线程
        finishCompletion();
    }
    return true;
}
```

