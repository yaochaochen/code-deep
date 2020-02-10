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