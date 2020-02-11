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
  - Netty 如何支持主从 Reactor 模式

    ```java
    public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup) {
        super.group(parentGroup);
        if (this.childGroup != null) {
            throw new IllegalStateException("childGroup set already");
        }
        this.childGroup = ObjectUtil.checkNotNull(childGroup, "childGroup");
        return this;
    }
    ```

  - 为什么说 Netty 的 main reactor 大多说并不是用到一个线程组，只能线程组里面一个

    

  - Netty 给 Channel 分配 NIO event loop 的规则是什么

    ```java
    public EventExecutor next() {
        return executors[Math.abs(idx.getAndIncrement() % executors.length)];
    }
    ```

    ```java
    public EventExecutor next() {
        return executors[idx.getAndIncrement() & executors.length - 1];
    }
    ```

  - 通用模式的 NIO 实现多路复用器是怎么跨平台的

    ```java
    public static SelectorProvider provider() {
        synchronized (lock) {
            if (provider != null)
                return provider;
            return AccessController.doPrivileged(
                new PrivilegedAction<>() {
                    public SelectorProvider run() {
                            if (loadProviderFromProperty())
                                return provider;
                            if (loadProviderAsService())
                                return provider;
                            provider = sun.nio.ch.DefaultSelectorProvider.create();
                            return provider;
                        }
                    });
        }
    }
    ```

## Netty 解决 粘包 半包问题

- 什么是粘包和半包
- 为什么 TCP 应用中会出现粘包和半包问题
- 解决粘包和半包问题的几种常用方法
- Netty 对三种常用封帧方法支持
- 解读 Netty 处理粘包 半包的源码

