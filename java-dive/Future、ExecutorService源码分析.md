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
// mayInterruptRunning 为 false 表示不能打断你直接返回 
public boolean cancel(boolean mayInterruptIfRunning) {
  //返回线程是否已经被取消了，true 表示已经被取消
  // 如果线程已经运行结束， isCancelled 和 isDone 返回都是 true
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
            } finally { // final state
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        finishCompletion();
    }
    return true;
}
```