# Netty 内存使用

## 目标

- 内存占用少(空间)
- 应用速度快(时间)

对 Java 而言 减少 Full GC 的 SWT (Stop the world) 时间

## 减少对象本身大小

例1:

| 基本类型 | 包装类  |
| -------- | ------- |
| byte     | Byte    |
| short    | Short   |
| int      | Integer |

包装类包含有对象本身+头部信息 占用空间大于基本类型



例2 应该定义类成员变量的不要定义为实例变量：

- 一个类 -> 一个类变量
- 一个实例 ->一个实例变量
- 一个类 -> 多个实例
- 实例越多，浪费越多

例3：Netty 中结合前两者

io.netty.channel.ChannelOutboundBuffer#incrementPendingOutboundBytes(long, boolean)

统计待写的请求的字节数

AtomicLong ->volatile long + static AtomicLongFiledUpdater

## 对分配内存进行预估

例1 对于可以预知的固定的 size 的 HashMap避免扩容

可以提前计算好初始 size 或者直接使用

com.google.common.collect.Maps#newHashMapWithExpectedSize

例2 Netty 根据接收到的数据动态调整 guess 下个要分配的 Buffer 的大小

io.netty.channel.AdaptiveRecvByteBufferAllocator

##  Zero-Copy

-  使用逻辑组合，代替实际复制

CompositeByteBuf：

io.netty.handler.codec.ByteToMessageDecoder#COMPOSITE_CUMULATOR

- 调用 JDK 的 zero-copy