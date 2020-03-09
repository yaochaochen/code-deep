# Netty 锁机制

## 分析同步问题的核心三要素

- 原子性: 并无一气呵成岂能无懈可击
- 可见性: 你做的改变 别人看不到
- 有序性: 不按套路出牌

## 锁的分类

- 对锁的态度: 乐观锁(java.util.concurrent 包中的原子类) 与悲观锁 (Synchronized)
- 等待锁的人是否公平: 公平锁 new ReentrantLock(true) 与非公平锁 new ReentrantLock()
- 是否可以共享: 共享锁与非独享锁 ReadWriteLock, 其读锁是共享锁 其写锁是独享锁

## 在意锁的对象和范围 -> 减少粒度

例如 初始化的 Channel(io.bootstrap.ServerBootstrap#init)

Sysnchronized method -> Synchronized block

例: 统计待发送的字节数(io.netty.channel.ChannelOutboundBuffer)

AtomicLong -> volatile long + AtomicLongFiledUpdater

### Atomic long VS long :

前者是一个对象， 包含对象头 (object header) 以用来 保存 hashcode lock 等信息，32位系统占用8字节 64位系统占16字节 所以在64位系统情况下 

- volatile long = 8 byte
- AtomicLong = 8 bytes (volatile long) + 16bytes(对象头) + 8bytes(引用) = 32bytes

至少节约24字节

结论:

Atomic * objects -> volatile primary type +static atomic*FiledUpdater

## 注意锁的速度 -> 提高并发性

例1 记录内存分配字节数等功能用到的 LongCounter (io.netty.util.internal.PlatformDependent#newLongCounter())

高并发时: java.util.concurrent.atomic.AtomicLong -> java.util.concurrent.atomic.LongAdder

例2: 曾经根据不同情况选择不同的并发包实现 JDK <1.8考虑

ConcurrentHashMapV8(ConcurrentHashMap 在 JDK 8 中的版本现在)



## 不同场景选择不同的并发包->因需而变

例1 关闭和等待关闭时间执行器 (Event Excutor)

Object.wait/notify -> CountDownLatch

io.netty.util.concurrent.SingleThredEventExector#threadLock

例2 Nio Event loop 中负责存储 task 的 Queue 

Jdk LinkedBlockingQueue(MPMC) -> jctools MPC

## Netty 应用场景下: 局部串行 + 整体并行> 一个队列+多个线程模式

- 降低用户开发难度 逻辑简单 提升处理性能
- 避免锁带来的上线文切换和并发保护等额外开销

避免用锁用   ThreadLocal 来避免资源争用 例如 Netty 轻量级的线程池实现

io.netty.util.Recycler#threadLocal