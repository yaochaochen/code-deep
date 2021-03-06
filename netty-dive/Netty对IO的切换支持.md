-  什么是经典的三种 I/O 模式

| 排队打饭模式       | BIO(阻塞 I/O)   | <JDK1.4  |
| ------------------ | --------------- | -------- |
| 点单、等待被叫模式 | NIO(非阻塞 I/O) | JDK1.4后 |
| 包厢模式           | AIO (异步I/O)   | JDK1.7   |

- 阻塞与非阻塞

  - 数据就绪前要不要等待

  - 阻塞:没有数据传过来时，读取会阻塞直到有数据；缓冲区满时，写操作也会阻塞

    非阻塞遇到这些情况，都会直接返回。

- 同步与异步

  - 数据就绪后，数据谁来操作完成?
  - 数据就绪后需要自己去读取是同步 数据就绪直接读好再回调给程序叫异步

- Netty 对三种 I/O模式的支持

  | ~~BIO->OIO（Deprecated）~~         | NIO(Common)            | ~~AIO(Removed)~~           |
  | ---------------------------------- | ---------------------- | -------------------------- |
  | ~~ThreadPerChannelEventLoopGroup~~ | NioEventLoopGroup      | ~~AioEventLoopGroup~~      |
  | ~~ThreadPerChannelEventLoop~~      | NioEventLoop           | ~~AioEventLoop~~           |
  | ~~OioServerSocketChannel~~         | NioServerSocketChannel | ~~AioServerSocketChannel~~ |
  | ~~OioSocketChannel~~               | NioSocketChannel       | ~~AioSocketChannel~~       |

  

- 为什么 Netty 仅支持 NIO

  - 为什么不建议阻塞 I/O (BIO/OIO)
    - 连接数高的情况下:阻塞->消耗资源 效率低
  - 为什么删除已经做好的 AIO 支持
    - Windows 实现成熟，但是很少用来做服务器
    - Linux 常用做服务器 但是AIO实现不成熟
    - Linux 下 AIO相比 NIO 的性能提升不明显

- 为什么 Netty 有多种 NIO 的支持

  | COMMON                 | Linux                    | macOS/BSD                 |
  | ---------------------- | ------------------------ | ------------------------- |
  | NioEventLoopGroup      | EpollEventLoopGroup      | KQueueEventLoopGroup      |
  | NioEventLoop           | EpollEventLoop           | KQueueEventLoop           |
  | NioServerSocketChannel | EpollServerSocketChannel | KQueueServerSocketChannel |
  | NioSocketChannel       | EpollSocketChannel       | KQueueSocketChannel       |

  通用的 NIO 实现 在 Linux 下也是使用 epoll, 为什么自己单独实现？

  - Netty 暴露了更多  的可控参数 比如
    - JDK 的 NIO 默认实现是水平触发
    - Netty 是边缘触发（默认）和水平触发可切换
  - Netty 实现垃圾回收更少 性能更好 

- NIO 一定优于 BIO 吗

  -  BIO 代码更简单
  - 特定场景：比如连接数少

  

