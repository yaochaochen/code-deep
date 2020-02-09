- 什么是经典的三种 I/O 模式

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

- NIO 一定优于 BIO 吗？

- 源码解读 Netty 切换 I/O？

