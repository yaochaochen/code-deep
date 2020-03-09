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

