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

  - 同时发送两个请求 abc 和 def 可能会收到 abcdef (粘包) 或者 收到 ab c de f(半包) 

- 为什么 TCP 应用中会出现粘包和半包问题

  - 粘包问题
    - 发送方每次写入数据 < 套接字缓冲区大小
    - 接收方读取套接字缓冲区数据不够及时
  - 半包问题
    - 发送方写入数据 > 套接字缓冲区大小
    - 发送的数据 > MUT （最大传输单元）必须拆包
  - 根本问题
    - TCP 是流式协议，消息无边界

- 解决粘包和半包问题的几种常用方法

  - 找出消息的边界

    - TCP 连接改为短连接一个请求一个短连接

      建立连接到释放连接之间信息即为传输信息

    - 封装成帧

      | 方式                           | 寻找消息边界方式                        | 优点                           | 缺点                           | 推荐度 |
      | ------------------------------ | --------------------------------------- | ------------------------------ | ------------------------------ | ------ |
      | 固定长度                       | 满足固定长度即可                        | 简单                           | 空间浪费                       | 不推荐 |
      | 分割符                         | 分隔符之间                              | 空间不浪费 比较简单            | 内容本身出现分隔符需转义       | 推荐   |
      | 固定长度字段存个内容的长度信息 | 先解析固定长度获取长度 然后读取后续内容 | 精准定位用户数据内容不需要转义 | 长度理论有限，需要阈值最大长度 | 推荐   |
      | 其他方式                       | 每种都不同 例如JSON 可以看成{}是否成对  |                                |                                |        |

    

- Netty 对三种常用封帧方法支持

  | 支持                           | 解码                         | 编码                 |
  | ------------------------------ | ---------------------------- | -------------------- |
  | 固定长度                       | FixedLengthFrameDecoder      | 简单                 |
  | 分割符                         | DelimiterBasedFrameDecoder   | 简单                 |
  | 固定长度字段存个内容的长度信息 | LengthFiledBasedFrameDecoder | LengthFiledPrepender |

  

- 解读 Netty 处理粘包 半包的源码

  ```java
  protected void callDecode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
      try {
          while (in.isReadable()) {
              int outSize = out.size();
  
              if (outSize > 0) {
                  fireChannelRead(ctx, out, outSize);
                  out.clear();
  
                  // Check if this handler was removed before continuing with decoding.
                  // If it was removed, it is not safe to continue to operate on the buffer.
                  //
                  // See:
                  // - https://github.com/netty/netty/issues/4635
                  if (ctx.isRemoved()) {
                      break;
                  }
                  outSize = 0;
              }
  
              int oldInputLength = in.readableBytes();
              decodeRemovalReentryProtection(ctx, in, out);
  
              // Check if this handler was removed before continuing the loop.
              // If it was removed, it is not safe to continue to operate on the buffer.
              //
              // See https://github.com/netty/netty/issues/1664
              if (ctx.isRemoved()) {
                  break;
              }
  
              if (outSize == out.size()) {
                  if (oldInputLength == in.readableBytes()) {
                      break;
                  } else {
                      continue;
                  }
              }
  
              if (oldInputLength == in.readableBytes()) {
                  throw new DecoderException(
                          StringUtil.simpleClassName(getClass()) +
                                  ".decode() did not read anything but decoded a message.");
              }
  
              if (isSingleDecode()) {
                  break;
              }
          }
      } catch (DecoderException e) {
          throw e;
      } catch (Exception cause) {
          throw new DecoderException(cause);
      }
  }
  ```

```java
protected final void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
    Object decoded = decode(ctx, in);
    if (decoded != null) {
        out.add(decoded);
    }
}
```

```java
protected Object decode(
        @SuppressWarnings("UnusedParameters") ChannelHandlerContext ctx, ByteBuf in) throws Exception {
    if (in.readableBytes() < frameLength) {
        return null;
    } else {
        return in.readRetainedSlice(frameLength);
    }
}
```

```java
* <h3>2 bytes length field at offset 1 in the middle of 4 bytes header,
*     strip the first header field and the length field, the length field
*     represents the length of the whole message</h3>
*
* Let's give another twist to the previous example.  The only difference from
* the previous example is that the length field represents the length of the
* whole message instead of the message body, just like the third example.
* We have to count the length of HDR1 and Length into <tt>lengthAdjustment</tt>.
* Please note that we don't need to take the length of HDR2 into account
* because the length field already includes the whole header length.
* <pre>
* lengthFieldOffset   =  1
* lengthFieldLength   =  2
* <b>lengthAdjustment</b>    = <b>-3</b> (= the length of HDR1 + LEN, negative)
* <b>initialBytesToStrip</b> = <b> 3</b>
*
* BEFORE DECODE (16 bytes)                       AFTER DECODE (13 bytes)
* +------+--------+------+----------------+      +------+----------------+
* | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
* | 0xCA | 0x0010 | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
* +------+--------+------+----------------+      +------+----------------+
* </pre>
```