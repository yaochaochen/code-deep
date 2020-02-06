## 为什么舍近求远: 不直接用 JDK NIO

### Netty 做得更好

- 支持常用应用层协议
- 解决传输问题: 粘包、半包现象
- 支持流量整形
- 完善的断链 idle等异常处理等

- Netty 做得更好之一：规避 JDK NIO bug：
  - 实例1： 经典的 epoll bug : 异常唤醒空转导致 CPU 100%

    JDK-6670302 （se） NIO selector wakes up with 0 selected keys inifinitely [Inx 2.4]

    Netty 解决之道: 检测 问题发生，然后处理

  - 实例2 ：IP_TOS 参数(IP 包的优先级和 Qos 选项) 使用时抛出异常

    java.lang.AssertionError： Option not found

    Netty 解决之道 遇到问题绕路走  

- Netty 做的更好之二： API更友好更强大
  - JDK 的 NIO 一些 API 不够友好 功能薄弱 例如 ByteBuffer -> Netty's ByteBuf
  - 除了 NIO 外 也提供了其他一些增强 Threadlocal ->Netty's FastThredLocal 