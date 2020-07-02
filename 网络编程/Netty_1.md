### Netty

#### Netty 介绍

1. Netty 是为了快速开发 可维护的 高性能、高可扩展的 网络服务器 和 客户端程序 而提供的异步事件驱动的基础框架和工具。

   即：Netty 是一个 Java NIO 客户端/服务器框架。基于 Netty ，可以快速轻松的开发网络服务器和客户端的应用程序。

##### Netty Channel

1. Netty 对 Java 中的 NIO 和 面向流的OIO（Old-IO：传统的阻塞式 IO）的 Channel 通道组件进行了再封装，使其他们都能支持多种通信协议。

   即：Netty 中的每一种协议的通道，都有 NIO（异步 IO） 和 OIO（阻塞式IO） 两个版本。

2. Netty 中常见的类型通道：

   + NIO

     ```
     NioSocketChannel：异步非阻塞 TCP Socket 传输通道（常用）
     NioServerSocketChannel：异步非阻塞 TCP Socket 服务器端监听通道（常用）
     NioDatagramChannel：异步非阻塞的 UDP 传输通道
     NioSctpChannel：异步非阻塞 Sctp 传输通道
     NioSctpServerChannel：异步非阻塞 Sctp 服务器端监听通道
     ```

   + OIO

     ```
     OioSocketChannel：异步非阻塞 TCP Socket 传输通道
     OioServerSocketChannel：异步非阻塞 TCP Socket 服务器端监听通道
     OioDatagramChannel：异步非阻塞的 UDP 传输通道
     OioSctpChannel：异步非阻塞 Sctp 传输通道
     OioSctpServerChannel：异步非阻塞 Sctp 服务器端监听通道
     ```

3. Netty 的 NioSocketChannel 内部组合了一个 Java NIO 的 SelectableChannel 成员。

   通过这个 Java NIO 通道，Netty 的 NioSocketChannel 通道上的 IO 操作，最终都会落地到 Java NIO 的 SelectableChannel 底层通道上。

   <img src="image\Netty 通道.jpg" style="zoom:80%;" />

##### Netty Reactor

+ 在 反应器模式中，一个反应器（或者 SubReactor 子反应器）会负责一个 事件处理线程，通过 Selector 选择器不断查询注册过的事件（选择键 SelectionKey）。如果查询到 IO 事件，则将其分发给 Handler 业务处理器。

+ 根据 Channel 通道类型的不同，Netty 实现了多种 反应器。

  `NioSocketChannel` 对应的 Netty反应器为 `NioEventLoop`

  ![](image\NioEventLoop 反应器.jpg)

  NioEventLoop 的 Thread、Selector 两个成员变量表明：一个反应器拥有一个 线程，负责一个 Java NIO Selector 选择器的 IO 事件轮询。

+ 注意：

  EventLoop 与 Channel 是一对多的关系：理论上，一个反应器可以注册成千上万个通道。

+ Netty 中的选择器（Selector）会监控事件，并将事件分发给对应的 Handler。

  可通选择器监控的 通道IO事件类型：

  1. SelectionKey.OP_READ：可读
  2. SelectionKey.OP_WRITE：可写
  3. SelectionKey.OP_CONNECT：连接
  4. SelectionKey.OP_ACCEPT：接收

+ 注意：

  通道和处理器 之间是多对多的关系：一个处理器可以绑定到多个通道，处理多个通道事件；一个 通道也可以被多个 处理器处理。

##### Netty Handler

+ Netty 中 处理器的基础接口为 ChannelHandler，该接口有两大类实现类：

  1. ChannelInboundHandler：通道入站处理器
  2. ChannelOutboundHandler：通道出站处理器

+ 入站处理

  + Netty的 入站处理，不仅仅是 OP_READ 输入事件的处理，还包括 从通道底层触发，由 Netty 通过层层传递，调用 ChannelInboundHandler 通道入站处理器进行的某个处理。**【？？】**

  + 例如：以底层的 Java NIO 中的 OP_READ 输入事件为例

    在通道中发生了 OP_READ 事件后，会被 EventLoop 查询到，然后分发给 ChannelInboundHandler 通道入站处理器，调用它的 入站处理方法 read，read 处理方法可以从 通道中读取数据。

+ 出站处理

  + Netty的出站处理，包括了 Java NIO 的 OP_WRITE 事件。**【？？】**

    注意：OP_WRITE 事件是 Java NIO 底层的概念，Netty 的出站处理是应用层维度的。

  + 出站处理具体是指：从 ChannelOutboundHandler 通道出战处理器 到 通道的某次 IO 操作。

  + 例如：

    在应用程序完成业务处理后，可以通过 ChannelOutboundHandler 通道出站处理器将处理结果写入底层通道，调用 write 方法将数据写到底层通道！

+ 入站 和 出站处理器都有默认的实现：

  + 入站处理器 ChannelInboundHandler ：ChannelInboundHandlerAdapter 通道入站处理器适配器
  + 出站处理器 ChannelOutboundHandler ：ChannelOutboundHandlerAdapter 通道出站处理器适配器

  两个默认实现 分别实现了 入站操作 和出站操作的基本功能。分别继承这两个通道适配器，开发者就可以实现自己的业务处理器。
  
+ 由于 Netty 中的 通道和Handler 之间是 多对多的关系，为了组织好各自的绑定关系，Netty 设计了一个特殊的组件：ChannelPipeline

##### Netty Pipeline

+ 通道流水线 将绑定到一个 通道的多个 Handler 处理器实体 链接在一起，形成一条流水线（双向链表结构）。所有的 Handler 实例被封装成一个阶段，加入到了 ChannelPipeline 中。

+ 申明：

  一个 Netty 通道拥有一条 Handler 处理器流水线（组合的方式：成员名称为 pipeline）

+ 流水线 Handler 处理顺序：

  每一个来自通道的 IO 事件，都会进入一次 ChannelPipeline 通道流水线。在进入第一个 Handler 处理器后，这个 IO 事件将按照既定的次序，在流水线上不断的向后流动，流向下一个 Handler。

  + 入站事件：从前往后流动，只会且只能从 Inbound 入站处理器类型的 Handler 流过
  + 出站事件：从后往前流动，只会且只能从 Outbound 出站处理器类型的 Handler 流过

  <img src="image\流水线.jpg" style="zoom:80%;" />

##### Netty Bootstrap

+ Bootstrap 类是 Netty 提供的一个 便利的 （组件组装）工厂类，可以通过它来完成 Netty 的客户端或者服务端的 Netty 组件的组装 及 Netty 程序的初始化。（你可以不用 Bootstrap 但是你要承受麻烦）

+ AbstractBootstrap 是基础类库，有 Bootstrap 和 ServerBootstrap 两大实现类，两种启动类的配置 和 使用方法大致相同。

  + Bootstrap：客户端专用
  + ServerBootstrap：服务端专用

+ **父子通道** 概念：

  + 理论上：操作系统底层的 socket 描述符分类两类：

    1. 连接监听类型

       ```
       连接监听类型的 socket 描述符，放在服务器端，它负责接收客户端的套接字连接。
       在服务端，一个 “连接监听类型” 的 socket 描述符可以接受成千上万的 传输类型的 socket 描述符。（1024个？）
       ```

    2. 传输数据类型

       ```
       数据传输类的 socket 描述符负责传输数据。
       同一条 TCP 的 Socket 传输链路，在服务器和客户端，都分别会有一个 与之相对应的 数据传输类型的 socket 描述符
       ```

  + 在 Netty 中

    NioServerSocketChannel（异步非阻塞的服务器端通道）封装的底层 socket 是 “连接监听类型” 的描述符

    NioSocketChannel （异步非阻塞 TCP Socket 传输通道） 封装的底层 socket 是 “传输数据类型” 的描述符

    

  + Netty 将负责 服务器连接监听和接受 的 NioServerSocketChannel 称为 父通道；将 负责接收数据的 NioSocketChannel 称为子通道。

    NioServerSocketChannel 和 NioSocketChannel 统称为 父子通道。

+ **EventLoopGroup 线程组** 概念：

  + Netty 中采用了 多线程的反应器，就是通过 EventLoopGroup 实现的。

  + 对于 Netty 而言，一个 NioEventLoop 子反应器独自享有一个线程，同时拥有一个 Java NIO 选择器；多个 EventLoop 线程 就组成了一个 EventLoopGroup 线程组。

  + Netty 应用开发并不会直接使用 EventLoop，而是由 EventLoopGroup 线程组，进行多线程的 IO 事件查询和分发。

    ```
    EventLoopGroup 构造方法中 有一个指定内部线程数的 参数，在构造器初始化时，会按照传入的线程数量，在内部构造多个 Thread 线程 和多个 EventLoop 子反应器（一个线程对应一个 EventLoop 子反应器）。
    
    如果创建 EventLoopGroup 时没有指定 线程数 或者指定的线程数为 0，则 Netty 默认将内部线程数设置为 2*最大可用 CPU 处理器数量
    ```

  + 在服务器端一般有两种独立的 反应器

    ```
    一种负责 新连接的监听和接受
    一种负责 IO 事件的处理
    ```

    因此，在 Netty 服务器程序中就对应两个 EventLoopGroup 线程组。

    + 负责新连接的 监听和接受 的 EventLoopGroup：

      会查询父通道的 IO 事件，称为 “Boss” 线程组

    + 负责 IO 事件的处理的 EventLoopGroup：

      会查询所有子通道的 IO 事件，并执行 Handler 处理器处理事件