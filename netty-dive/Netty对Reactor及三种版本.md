-  线程 
- 接入连接
- 请求 
  1. Reactor 单线程
  2. Reactor 多线程
  3. 主从 Reactor 多线程模式
- 业务处理
- 响应
- 断链

- 什么是 Reactor及三种版本

| BIO               | NIO     | AIO      |
| ----------------- | ------- | -------- |
| Thread-Connection | Reactor | Proactor |

Reactor 是一种开发模式，模式的核心流程:

注册事件->扫描事件->处理事件

| clinet/Server | SocketChannel/ServerSocketChannel | OP_ACCEPT | OP_CONNECT | OP_WRITER | OP_READ |
| ------------- | --------------------------------- | --------- | ---------- | --------- | ------- |
| client        | SocketChannel                     |           | Y          | Y         | Y       |
| server        | ServerSocketChannel               | Y         |            |           |         |
| server        | SocketChannel                     |           |            | Y         | Y       |

- Netty 中使用 Reactor 模式

  | Reactor 单线程模式        | EventLoopGroup eventGroup = new NioEventLoopGroup(1)         |
  | ------------------------- | ------------------------------------------------------------ |
  | 非主从 Reactor 多线程模式 | EventLoopGroup eventGroup = new NioEventLoopGroup()          |
  | 主从 Reactor 多线程模式   | EventLoopGroup bossGroup = new NioEventLoopGroup() EventLoopGroup workerGroup = new NioEventLoopGroup() |

  