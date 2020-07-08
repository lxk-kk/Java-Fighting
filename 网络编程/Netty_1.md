### Netty

##### Netty 介绍

1. Netty 是为了快速开发 可维护的 高性能、高可扩展的 网络服务器 和 客户端程序 而提供的异步事件驱动的基础框架和工具。

   即：Netty 是一个 Java NIO 客户端/服务器框架。基于 Netty ，可以快速轻松的开发网络服务器和客户端的应用程序。

##### Netty Channel

###### 介绍

1. 在 Netty 中，通道代表着网络连接，负责同对端进行网络通信。通过通道，可以写入数据到对端，也可以从对端读取数据。

2. Netty 对 Java 中的 NIO 和 面向流的OIO（Old-IO：传统的阻塞式 IO）的 Channel 通道组件进行了再封装，使其他们都能支持多种通信协议。

   即：Netty 中的每一种协议的通道，都有 NIO（异步 IO） 和 OIO（阻塞式IO） 两个版本。

###### AbstractChannel

1. 通道抽象类 AbstractChannel

   ```java
   protected AbstractChannel(Channel parent, Integer id) {
       if (id == null) {
           id = allocateId(this);
       } else {
           if (id.intValue() < 0) {
               throw new IllegalArgumentException("id: " + id + " (expected: >= 0)");
           }
   
           if (allChannels.putIfAbsent(id, this) != null) {
               throw new IllegalArgumentException("duplicate ID: " + id);
           }
       }
   
       // 父通道
       this.parent = parent;
       // 通道 id ？
       this.id = id;
       // 底层 IO 操作类（NIO、OIO）
       this.unsafe = this.newUnsafe();
       // 通道流水线：每个通道都拥有一条流水线
       this.pipeline = new DefaultChannelPipeline(this);
       // 为 通道关闭任务 添加 异步回调监听器
       this.closeFuture().addListener(new ChannelFutureListener() {
           public void operationComplete(ChannelFuture future) {
               AbstractChannel.allChannels.remove(AbstractChannel.this.id());
           }
       });
   }
   ```

   ```
   AbstractChannel 中的 parent 属性，表示该通道的父通道：
   对于 连接监听通道而言（例如：NioServerSocketChannel），其父通道为 null
   对于 每条传输通道而言（例如：NioSocketChannel），其 父通道 为接收到该连接的服务器连接监听通道。
   ```

2. Netty 中常见的类型通道：

   + 几乎所有的通道是实现类都继承了 AbstractChannel 抽象类

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

3. AbstractChannel 常见方法

   ```java
   // 连接远程服务器：在客户端的传输通道使用
   ChannelFuture connect(SocketAddress address);
   // 绑定监听地址，开始监听新的客户端连接：在服务器的新连接监听和接收通道使用
   ChannelFuture bind(SocketAddress address);
   // 关闭通道连接
   ChannelFuture close();
   
   // 读取通道数据，并且启动入栈处理：方法返回通道自身 用于 链式调用
   // 即：从 内部的 Java NIO Channel 通道读取数据，然后启动内部的 pipeline 流水线
   Channel read();
   
   // 启程出栈流水处理，将处理后的最终数据写入底层 Java NIO 通道
   ChannelFuture write(Object o);
   
   // 将缓冲区的数据立即写到对端。
   // 因为并不是每次调用 write 操作都是将数据直接写出到对端，write 大部分情况下仅仅写入数据到操作系统的缓冲区，而操作系统会根据缓冲区的情况，决定何时将数据写到对端
   // 执行 flush 方法立刻将缓冲区中的数据写到对端
   Channel flush();
   ```

   

###### NioSocketChannel

1. Netty 的 NioSocketChannel 内部组合了一个 Java NIO 的 SelectableChannel 成员。

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

###### 介绍

+ Reactor 模式中，反应器查询到 IO 事件后，将其分发给 Handler 业务处理器，由 Handler 完成 IO 操作业务处理。

+ 整个 IO 处理环节包括：

  <img src="image\Handler IO处理环节.jpg" style="zoom:80%;" />

  + 从通道读取数据包（read）、数据包解码（decode）、业务处理（compute）、目标数据编码（encode）、数据包写到通道（send）

  + 上述过程中：*从通道读取数据包*、*由通道发送数据包到对端* 两个环节由 Netty 底层负责完成，不需要用户程序干预。

  + 用户程序主要在 Handler 业务处理器中：

    Handler 涉及到：数据包解码、业务处理、目标数据编码、把数据包写到通道中（？）

###### ChannelHandler

+ Netty 中 处理器的基础接口为 ChannelHandler，该接口有两大类实现类：

  1. ChannelInboundHandler：通道入站处理器
  2. ChannelOutboundHandler：通道出站处理器

+ 入站 和 出站处理器都有默认的实现：

  默认实现 分别实现了 入站操作 和出站操作的基本功能。分别继承这两个通道适配器，开发者就可以实现自己的业务处理器。
  
+ 入站处理器 ChannelInboundHandler ：ChannelInboundHandlerAdapter 通道入站处理器适配器
  
  + 出站处理器 ChannelOutboundHandler ：ChannelOutboundHandlerAdapter 通道出站处理器适配器
  
+ 由于 Netty 中的 通道和Handler 之间是 多对多的关系，为了组织好各自的绑定关系，Netty 设计了一个特殊的组件：ChannelPipeline

###### ChannelInboundHandler

+ 通道入站处理器

  + 触发方向：自底向上，Netty 内部（通道）到 ChannelInboundHandler 入站处理器

  + Netty的 入站处理，不仅仅是`OP_READ`输入事件的处理，还包括 从通道底层触发，由`Netty`通过层层传递，调用`ChannelInboundHandler`通道入站处理器进行的某个处理。**【？？】**

    ```
    例如：以底层的 Java NIO 中的 OP_READ 输入事件为例
    
    在通道中发生了 OP_READ 事件后，会被 EventLoop 查询到，然后分发给 ChannelInboundHandler 通道入站处理器，调用它的 入站处理方法 read，read 处理方法可以从 通道中读取数据。
    ```

+ 常用 API

  ```
  API 调用概述：
  1. 通道完成某种状态后，Netty 会调用对应的方法(fire) 触发通道中对应的事件。
  2. 通道流水线感知到事件，并调用入站处理器中 对应的 处理方法 进行后续处理。
  ```

  + `channelRegistered`

    当通道注册完成后，Netty 会调用 fireChannelRegistered 触发通道注册事件。

    然后流水线工作：流水线中的 入站处理器，将会调用 channelRegistered 方法进行处理。

  + `channelActive`

    当通道激活完成后，Netty 会调用 fireChannelActive 触发通道激活事件。

    然后流水线工作：入站处理器 将调用 channelActive 方法进行处理。

  + `channelRead`

    当通道缓冲区可读时，Netty 会调用 fireChannelRead 触发通道可读事件。

    然后流水线工作：入站处理器 将调用 channelRead 方法进行处理

  + `channelReadComplete`

    当通道缓冲区 读完 时，Netty 会调用 fireChannelReadComplete 触发通道读完事件

    然后流水线工作：入站处理器 将调用 channelReadComplete 方法进行处理

  + `channelInactive`

    当 连接断开或者不可用 时，Netty 会调用 fireChannelInactive 触发连接不可用事件。

    然后流水线工作：入站处理器 将调用 channelnactive 方法进行处理。

  + `exceptionCaught`

    当通道处理处理过程中发生异常时，Netty 会调用 fireExceptionCaught 触发异常捕获事件。

    然后流水线工作：入站处理器会调用 exceptionCaught 方法进行处理。

    注意：exceptionCaught 方法是 ChannelHandler 声明的方法，入站和出站 处理器接口都会继承。

###### ChannelOutboundHandler

+ 通道出站处理器

  + 触发方向：自顶向下，从 ChannelOutboundHandler 出站处理器到 Netty 内部（通道）

    通过上层 Netty 通道 操作底层 Java IO 通道。

  + Netty的出站处理，包括了 Java NIO 的 OP_WRITE 事件。**【？？】**

    注意：OP_WRITE 事件是 Java NIO 底层的概念，Netty 的出站处理是应用层维度的。

  + 出站处理具体是指：从 ChannelOutboundHandler 通道出站处理器 到 通道的某次 IO 操作。

    ```
    例如：
    
    在应用程序完成业务处理后，可以通过 ChannelOutboundHandler 通道出站处理器将处理结果写入底层通道，调用 write 方法将数据写到底层通道！
    ```

+ 常用 API

  + `bind`

    监听地址（IP+端口）绑定：完成底层 Java IO 通道的 IP 地址绑定。

    如果使用 TCP 协议，则该方法用于 服务器端。

  + `connect`

    连接服务端：完成底层 Java IO 通道的服务端的连接操作。

    如果使用 TCP 协议，则该方法用于 客户端。

  + `write`

    写数据到底层：完成 Netty 通道向底层 Java IO 通道的数据写入操作。

    该方法仅仅触发，并非完成实际的数据写入。

  + `flush`

    刷新缓冲区的数据，将数据写到对端：将底层 缓冲区 的数据立刻写到对端。

  + `read`

    从底层读取数据：完成 Netty 通道从 Java IO 通道的数据读取。【？出站还要读取？】

  + `disConnect`

    断开服务器连接：断开底层 Java IO 通道的服务器端连接。

    如果使用 TCP 协议，则该方法主要用于 客户端。

  + `close`

    主动关闭通道：关闭底层通道。

    例如：服务器端的新连接监听通道。

###### ChannelInitializer

+ 通道初始化处理器

  在通道开始工作之前，向通道中的 流水线 装配 Handler。

+ **initChannel** 方法

  该方法是 ChannelInitializer 中的抽象方法，需要开发者自行实现，用于为 通道的流水线中装配 Handler

  ```
  在 父通道 调用 initChannel 方法时，会将新接收的通道作为参数，传递给 initCahnnel 方法，在 initChannel 方法中，为新连接的通道的流水线装配 Handler 业务处理器。
  ```

###### ChannelHandler 生命周期

+ 代码示例：以 ChannelInboundHandler 生命周期 为例！

  ```
  入站处理器的生命周期与入站处理器的生命周期大致相同，不同的时 出站处理器的业务处理（出站回调处理方法？）
  ```

  + ChannelInboundHandler  实现类

    ```java
    /**
     * @author 10652
     */
    public class InHandlerDemo extends ChannelInboundHandlerAdapter {
        @Override
        public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
            System.out.println("调用：channelRegistered()");
            super.channelRegistered(ctx);
        }
    
        @Override
        public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
            System.out.println("调用：channelUnregistered()");
            super.channelUnregistered(ctx);
        }
    
        @Override
        public void channelActive(ChannelHandlerContext ctx) throws Exception {
            System.out.println("调用：channelActive()");
            super.channelActive(ctx);
        }
    
        @Override
        public void channelInactive(ChannelHandlerContext ctx) throws Exception {
            System.out.println("调用：channelInactive()");
            super.channelInactive(ctx);
        }
    
        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
            System.out.println("调用：channelRead()");
            super.channelRead(ctx, msg);
        }
    
        @Override
        public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
            System.out.println("调用：channelReadComplete()");
            super.channelReadComplete(ctx);
        }
    
        @Override
        public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
            System.out.println("调用：handlerAdded()");
            super.handlerAdded(ctx);
        }
    
        @Override
        public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
            System.out.println("调用：handlerRemoved()");
            super.handlerRemoved(ctx);
        }
    }
    ```

  + Test

    ```java
    public class Test {
        public static void main(String[] args) throws InterruptedException {
            // 入站处理器 Demo
            final InHandlerDemo inHandlerDemo = new InHandlerDemo();
            // 通道初始化器：在通道工作之前为通道配置 处理器
            // EmbeddedChannel ：嵌入式通道（模拟）
            ChannelInitializer initializer = new ChannelInitializer<EmbeddedChannel>() {
                @Override
                protected void initChannel(EmbeddedChannel embeddedChannel) throws Exception {
                    System.out.println("调用：initChannel()：为通道流水线初始化 handler");
                    embeddedChannel.pipeline().addLast(inHandlerDemo);
                }
            };
            
            // 创建通道
            System.out.println("--------------step 0-----------------");
            EmbeddedChannel channel = new EmbeddedChannel(initializer);
    
            ByteBuf buf = Unpooled.buffer();
            buf.writeInt(1);
            Thread.sleep(50);
    
            // 两次模拟入站，写入入站数据包
            System.out.println("--------------step 1-----------------");
            channel.writeInbound(buf);
            channel.flush();
            channel.writeInbound(buf);
            channel.flush();
            Thread.sleep(50);
    
            // 通道关闭
            System.out.println("--------------step 2-----------------");
            channel.close();
        }
    }
    ```

  + 测试结果

    ```
    --------------step 0-----------------
    调用：initChannel()：为通道流水线初始化 handler
    调用：handlerAdded()
    调用：channelRegistered()
    调用：channelActive()
    --------------step 1-----------------
    调用：channelRead()
    调用：channelReadComplete()
    调用：channelRead()
    调用：channelReadComplete()
    --------------step 2-----------------
    调用：channelInactive()
    调用：channelUnregistered()
    调用：handlerRemoved()
    ```

+ 解析：

  上述方法分类：

  1. 生命周期方法：

     通道创建时：handlerAdd、channelRegistered、channelActive 

     通道关闭时： channelInactive、channelUnregistered、handlerRemove 

  2. 入站回调方法：

     channelRead、channelReadComplete

+ 生命周期方法解析：

  ```
  1. handlerAdded ：当 handler 加入到通道流水线后，此方法被回调。即：ch.pipeline().addLast(hanlder) 后回调
  2. channelRegistered ：当通道成功绑定一个 NioEventLoop 线程后，会通过流水线回调所有 处理器handler 的 channelRegistered 方法。
  3. channelActive ：当通道激活成功后，会通过流水线回调所有业务处理器的 channelActive 方法。
    通道激活成功：即所有 handler 的添加、注册 等异步任务完成，并且 NioEventLoop 线程绑定异步任务完成。
    
  4. channelInactive ：当通道的底层连接已经不是 ESSTABLISH 状态，或者底层连接已经关闭时，会首先回调所有业务处理器的 channelInactive 方法。
  5. channelUnregistered ：通道 和 NioEventLoop 线程接触绑定，移除掉对这条通道的事件处理之后，回调所有 handler 的 channelUnregistered 方法。
  6. handlerRemoved ：最后 Netty 会移除掉通道上所有的 handler 并回调所有 handler 的 handlerRemoved 方法
  ```

+ 入站回调方法解析：

  ```
  1. channelRead ：有数据包入站，通道可读。
    流水线会启动入站处理器流程，从前向后，入站处理器的 channelRead 方法会依次回调。
  2. channelReadComplete ：流水线完成入站处理后，会从前向后，依次回调每个入站处理器的 channelReadComplete 方法，表示数据读取完毕。
  ```

##### Netty Pipeline

###### 介绍

+ 通道流水线 将绑定到一个 通道的多个 Handler 处理器实体 链接在一起，形成一条流水线（**双向链表**结构）。所有的 Handler 实例被封装成一个结点，加入到了 ChannelPipeline 中。

+ 声明：

  一个 Netty 通道拥有一条 Handler 处理器流水线（组合的方式：成员名称为 pipeline）

  Netty 是基于 **责任链模式** 设计的 处理器流水线，内部是一个 双向链表 结构，能够支持动态的添加和删除 Handler 业务处理器。

###### pipeline

+ 流水线 Handler 处理顺序：

  每一个来自通道的 IO 事件，都会进入一次 ChannelPipeline 通道流水线。在进入第一个 Handler 处理器后，这个 IO 事件将按照既定的次序，在流水线上不断的向后流动，流向下一个 Handler。

  + 入站事件：从前往后流动，只会且只能从 Inbound 入站处理器类型的 Handler 流过
  + 出站事件：从后往前流动，只会且只能从 Outbound 出站处理器类型的 Handler 流过

  <img src="image\流水线.jpg" style="zoom:80%;" />
  
+ *代码示例*

  ```java
  /**
   * @author 10652
   */
  public class PipelineTest {
      public static void main(String[] args) {
          testInHandle();
          System.out.println("-----");
          testOutHandle();
      }
      // 出站示例
      private static void testOutHandle() {
          ChannelInitializer<EmbeddedChannel> initializer = new ChannelInitializer<EmbeddedChannel>() {
              @Override
              protected void initChannel(EmbeddedChannel embeddedChannel) throws Exception {
                  embeddedChannel.pipeline().addLast(new OutHandleA());
                  embeddedChannel.pipeline().addLast(new OutHandleB());
                  embeddedChannel.pipeline().addLast(new OutHandleC());
              }
          };
          EmbeddedChannel channel = new EmbeddedChannel(initializer);
          ByteBuf buf = Unpooled.buffer();
          buf.writeInt(0);
          channel.writeOutbound(buf);
      }
      // 入站示例
      private static void testInHandle() {
          ChannelInitializer initializer = new ChannelInitializer<EmbeddedChannel>() {
              @Override
              protected void initChannel(EmbeddedChannel embeddedChannel) throws Exception {
                  embeddedChannel.pipeline().addLast(new InHandleA());
                  embeddedChannel.pipeline().addLast(new InHandleB());
                  embeddedChannel.pipeline().addLast(new InHandleC());
              }
          };
          EmbeddedChannel channel = new EmbeddedChannel(initializer);
          ByteBuf buf = Unpooled.buffer();
          buf.writeInt(1);
          channel.writeInbound(buf);
      }
      // 出站处理器 A、B、C
      static class OutHandleA extends ChannelOutboundHandlerAdapter {
          @Override
          public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
              System.out.println("out handle a");
              super.write(ctx, msg, promise);
          }
      }
  
      static class OutHandleB extends ChannelOutboundHandlerAdapter {
          @Override
          public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
              super.write(ctx, msg, promise);
              System.out.println("out handle b");
          }
      }
  
      static class OutHandleC extends ChannelOutboundHandlerAdapter {
          @Override
          public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
              System.out.println("out handle c");
              super.write(ctx, msg, promise);
          }
      }
      // 入站处理器 A、B、C
      static class InHandleA extends ChannelInboundHandlerAdapter {
          @Override
          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
              System.out.println("in handle a");
              super.channelRead(ctx, msg);
          }
      }
  
      static class InHandleB extends ChannelInboundHandlerAdapter {
          @Override
          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
              System.out.println("in handle b");
              super.channelRead(ctx, msg);
          }
      }
  
      static class InHandleC extends ChannelInboundHandlerAdapter {
          @Override
          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
              System.out.println("in handle c");
              super.channelRead(ctx, msg);
          }
      }
  }
  ```

  *测试结果*

  ```
  in handle a
  in handle b
  in handle c
  -----
  out handle c
  out handle a
  out handle b
  ```

+ **注意：**

  上述 处理器调用链（出站、入站），如果断链，则后续处理器结点都不会执行！

###### ChannelHandlerContext

+ ChannelHandlerContext：通道处理器上下文

  处理器都是以 双向链表的结构 保存在 pipeline 中的，而 ChannelHandlerContext 就是一个 Handler 结点

  ```java
  public class DefaultChannelPipeline implements ChannelPipeline {
      private static final String HEAD_NAME = generateName0(DefaultChannelPipeline.HeadContext.class);
      private static final String TAIL_NAME = generateName0(DefaultChannelPipeline.TailContext.class);
  
      final AbstractChannelHandlerContext head;
      final AbstractChannelHandlerContext tail;
      // ... ...
      // 如下默认 通道流水线 的初始化方法：
      protected DefaultChannelPipeline(Channel channel) {
          // ... ...
          this.tail = new DefaultChannelPipeline.TailContext(this);
          this.head = new DefaultChannelPipeline.HeadContext(this);
          this.head.next = this.tail;
          this.tail.prev = this.head;
      }
      
      final class TailContext extends AbstractChannelHandlerContext implements ChannelInboundHandler {
          TailContext(DefaultChannelPipeline pipeline) {
              super(pipeline, (EventExecutor)null, DefaultChannelPipeline.TAIL_NAME, DefaultChannelPipeline.TailContext.class);
              this.setAddComplete();
          }
          // ... ...
      }
  
      final class HeadContext extends AbstractChannelHandlerContext implements ChannelOutboundHandler, ChannelInboundHandler {
          HeadContext(DefaultChannelPipeline pipeline) {
              super(pipeline, (EventExecutor)null, DefaultChannelPipeline.HEAD_NAME, DefaultChannelPipeline.HeadContext.class);
              this.unsafe = pipeline.channel().unsafe();
              this.setAddComplete();
          }
      }
  }
  ```

+ handler 添加到 pipeline 时，会创建一个 通道处理器上下文 ChannelHandlerContext，构建 上下文 双向链表。

  ```java
  public class DefaultChannelPipeline implements ChannelPipeline {
      // 添加 handler
      public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
          AbstractChannelHandlerContext newCtx;
          synchronized(this) {
              checkMultiplicity(handler);
              newCtx = this.newContext(group, this.filterName(name, handler), handler);
              // 创建新结点，并添加到 链表中（通道处理器上下文）
              this.addLast0(newCtx);
              if (!this.registered) {
                  newCtx.setAddPending();
                  this.callHandlerCallbackLater(newCtx, true);
                  return this;
              }
  
              EventExecutor executor = newCtx.executor();
              if (!executor.inEventLoop()) {
                  this.callHandlerAddedInEventLoop(newCtx, executor);
                  return this;
              }
          }
  
          this.callHandlerAdded0(newCtx);
          return this;
      }
      
      // 创建新结点，并添加到 链表中（通道处理器上下文）
      private void addLast0(AbstractChannelHandlerContext newCtx) {
          AbstractChannelHandlerContext prev = this.tail.prev;
          newCtx.prev = prev;
          newCtx.next = this.tail;
          prev.next = newCtx;
          this.tail.prev = newCtx;
      }
  }
  ```

+ ChannelHandlerContext 的两类方法：

  1. 获取上下文关联的 Netty 组件：通道、流水线、上下文内部 Handler 处理器 等实例
  2. 入站和出站方法

+ 通道（Channel）、流水线（ChannelPipeline）、处理器（ChannelHandler）、通道处理器上下文（ChannelHandlerContext）：

  **一条 Channel 中拥有一条 ChannelPipeline，流水线以 双向链表 表示，每个结点是一个 ChannelHandlerContext，而 ChannelHandlerContext 中封装的是  ChannelHandler。**

###### Pipeline 截断

+ 执行下一个 入站处理器 handler 的方式：

  ```
  1. super.channelXXX( ctx, msg );
  2. ctx.fireChannelXXX( );
  ```

  因此：只要在处理器执行的过程中，如果条件不满足就不调用上述方法，此时后续 handler 将不会被截断

+ 出站处理流程只要执行，就不能被截断，强行接断时会导致 Netty 异常。**【？测试的时候没有 异常】**

  如果条件不满足，可以不启动出站处理。

###### Pipeline 热插拔

+ pipeline 中定义了 handler 的增删方法。

  由于双向链表的 上下文结点 ChannelHandlerContext 中保持了 本 流水线的实例，因此在 handler 链工作时，能够通过 ChannelHandlerContext 结点动态删除 handler！

  ```java
  abstract class AbstractChannelHandlerContext implements ChannelHandlerContext, ResourceLeakHint {
      private final DefaultChannelPipeline pipeline;
      // ... ...
  
      // 参数中的 pipeline 就是本结点所在的 流水线 实例。
      AbstractChannelHandlerContext(DefaultChannelPipeline pipeline, EventExecutor executor, String name, Class<? extends ChannelHandler> handlerClass) {
          // ... ...
          this.pipeline = pipeline;
      }
  }
  ```

+ 使用方法：

  ```
  ctx.pipeline().remove(this); // 删除本结点
  ctx.pipeline().addLast(...); // 添加
  ... ...
  ```

+ 为什么需要 删除 呢？

  满足某条件后，需要删除本结点，初始化时只需要执行一次 handler

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

      会查询所有子通道的 IO 事件，并执行 Handler 处理器处理事件，称为 “worker” 线程组
  
+ **Bootstrap 启动流程**

  + Bootstrap 的启动流程 就是 Netty 组件的组装、配置 以及 Netty 服务器/客户端的启动过程

  + 分为 8 个步骤，示例：服务器端启动器的使用

    ```java
    // 首先：创建一个 服务器端 的启动器
    ServerBootstrap bootstrap = new ServerBootstrap();
    ```

    1. 配置 **反应器线程组**（EventLoopGroup）

       ```java
       // 创建 反应器线程组：boss 线程组
       EventLoopGroup bossLoopGroup = new NioEventLoopGroup(1);
       
       // 创建 反应器线程组：worker 线程组
       EventLoopGroup workerLoopGroup = new NioEventLoopGroup();
       
       // 将线程组 绑定到 启动器中
       bootstrap.group(bossLoopGroup, workerLoopGroup);
       ```

       + 注意：

         ```java
         bootstrap.group(boosLoopGroup, workerLoopGroup);
         // 这种方式一次性给 启动器配置 了两个线程组，一个负责监听和接受新连接，一个负责处理 处理 IO 事件。
         ```

         可以只为 启动器配置一个 线程组：

         ```java
         bootstrap.group(eventLoopGroup);
         // 此时，连接监听 IO 事件 和 数据传输 IO 事件可能被挤在了 同一个线程 中处理
         // 存在潜在的风险：新连接的接受 会被 耗时严重的数据传输或者业务处理 所阻塞。
         ```

       + 建议：设置成两个线程组的工作模式。

    2. 配置 **监听端口**

       ```java
       // 主要是设置 服务器的监听地址
       boostrap.localAddress( new InetSocketAddress( port ) );
       ```

    3. 配置 **通道 IO 类型**

       ```java
       // 是一个 父通道
       bootstrap.channel( NioServerSocketChannel.class );
       ```

       Netty 支持 Java 的 NIO 和 OIO（即：BIO 阻塞式 IO），如果需要配置 OIO 模型，则将 类型 替换为 `OioServerSocketChannel.class`

    4. 配置 **传输通道的通道选项**

       ```java
       // 设置是否开启 TCP 底层的心跳机制 【？？？】
       bootstrap.option( ChannelOption.SO_KEEPALIVE, true);
       bootstrap.option( ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);
       ```

       对于服务器 Bootstrap 而言：

       + `option`方法是为 父通道 接收 连接通道设置 一些选项**【？？？】**

       + 如果要给 子通道 设置一些通道选项，则需要使用`childOption()`方法

    5. 配置 **通道 Pipeline 流水线**

       ```java
       // 装配 子通道流水线
       bootstrap.childHandler( new ChannelInitializer<SocketChannel>(){
           // 有连接到达时 会创建一个 通道的子通道，并初始化
           protected void initChannel( SocketChannel ch ) throws Exception{
               // 向子通道流水线 添加一个 Handler 业务处理器
               ch.pipeline().addLast(new TestNettyHandler());
           }
       })
       ```

       为 通道配置 Handler 流水线：

       + 为 *父通道*  配置 Handler 流水线

         ```java
         // 如果 父通道有特殊的业务处理，则为父通道配置 ChannelInitializer 初始化器。
         bootstrap.handler( ChannelHandler handler );
         ```

         由于 父通道的业务逻辑固定：接收新连接后，创建并初始化子通道

         因此，父通道不需要特别的配置

       + 为 *子通道*  配置 Handler 流水线

         ```java
         bootstrap.childHandler( ChannelHandler childHandler );
         ```

         `childHandler`方法参数为`ChannelInitializer`通道初始化类的实例。

         当 父通道接收一个 连接，并成功创建一个 子通道之后，就会通过该`ChannelInitializer`实例初始化子通道。`ChannelInitializer`实例会调用`initChannel`方法向 子通道流水线 增加业务处理器。

       注意：

       ```
       ChannelInitializer 处理器有一个 泛型参数 SocketChannel，它代表需要初始化的通道类型，该类型需要和 启动器 中配置的通道类型一一对应。
       ```

    6. **绑定服务器 新连接 的监听端口**

       ```java
       // 绑定端口？通过调用 sync 同步方法阻塞直到绑定成功
       ChannelFuture channelFuture = bootstrap.bind().sync();
       ```

       `bootstrap.bind()`方法：返回一个端口 绑定 Netty 的异步任务`ChannelFuture`

       *这里并没有给 ChannelFuture 异步任务增加回调监听器，而是阻塞 ChannelFuture 异步任务，直到端口绑定任务执行成功。*

    7. **自我阻塞，直到通道关闭**

       ```java
       // 自我阻塞，直到通道关闭的异步任务结束
       ChannelFuture closeFuture = channelFuture.channel().closeFuture();
       closeFuture.sync();
       ```

       如果要阻塞 当前线程 直到通道关闭，可以使用 通道的 closeFuture() 方法 获取通道关闭的 异步任务。

       当通道被关闭时，closeFuture 实例的 sync() 方法才会返回。

    8. **关闭 EventLoopGroup**

       ```java
       // 释放所有资源，包括创建的反应器线程组
       workerLoopGroup.shutdownGracefully();
       bossLoopGroup.shutdownGracefully();
       ```

       ```
       关闭 Reactor 反应器线程组，同时会关闭内部的 subReactor 子反应器线程、Selector 选择器、轮询线程 以及 负责查询的所有子通道。子通道关闭后，会释放掉底层的资源，例如 TCP Socket 文件描述符等。
       ```

##### Netty ChannelOption

Netty 中的通道 可以通过 ChannelOption 设置一些列通道属性，常见如下的通道选项：

1. **SO_RCVBUF**, **SO_SNDBUF**

   TCP 参数：用于设置 TCP 的两个缓冲区大小

   ```
   每个 TCP Socket 在内核中都有一个 发送缓冲区和一个接收缓冲区
   
   TCP的 全双工模式 以及 滑动窗口 就依赖于这两个独立的缓冲区及其填充的状态。
   ```

2. **TCP_NODELAY**

   TCP参数：用于设置`Nagle`算法的启用，表示是否立即发送数据（Netty默认为True，而操作系统默认为False）

   ```
   Nagle 算法：将小的碎片数据连接成更大的报文（或数据包）来最小化所发送报文的数量，如果需要发送一些较小的报文，则需要禁用该算法。Netty默认禁用该算法，从而最小化报文传输的延时。
   
   注意：TCP_NODELAY 参数的值，与是否开启 Nagle 算法相反
   
   如果要求高实时性，有数据发送时就立刻发送，就设置为true，如果需要减少发送次数和减少网络交互次数，就设置为false。
   ```

3. **SO_KEEPALIVE**

   TCP参数：表示底层TCP协议的心跳机制（true为连接保持心跳，默认值为false）

   ```
   保持心跳连接时，TCP 会主动探测 空闲连接的有效性。
   注意：
   	默认的心跳间隔是7200s即2小时。Netty默认关闭该功能。
   ```

4. **SO_REUSEADDR**

   TCP参数：是否可以地址复用（默认值为 false）

   使用该参数的 四种 情况：

   + 当有一个有相同本地地址和端口的socket1处于TIME_WAIT状态时，而我们希望启动的程序的socket2要占用该地址和端口。例如在重启服务且保持先前端口时。
   + 有多块网卡或用IP Alias技术的机器在同一端口启动多个进程，但每个进程绑定的本地IP地址不能相同。
   + 单个进程绑定相同的端口到多个socket（套接字）上，但每个socket绑定的IP地址不同。
   + 完全相同的地址和端口的重复绑定。但这只用于UDP的多播，不用于TCP。

5. **SO_LINGER**

   TCP参数：表示关闭socket的延迟时间，默认值为-1，表示禁用该功能。

   + -1 ：socket.close()方法立即返回，但操作系统底层会将发送缓冲区全部发送到对端。
   + 0 ：socket.close()方法立即返回，操作系统放弃发送缓冲区的数据，直接向对端发送RST包，对端收到复位错误。
   + 非0整数值 ：调用socket.close()方法的线程被阻塞，直到延迟时间到来、发送缓冲区中的数据发送完毕，若超时，则对端会收到复位错误。

6. **SO_BACKLOG**

   TCP参数：表示服务器端接收连接的队列长度，如果队列已满，客户端连接将被拒绝。

   参数默认值：在Windows中为200，其他操作系统为128。

   如果连接建立频繁，服务器处理新连接较慢，可以适当调大这个参数．

7. **SO_BROADCAST**

   TCP参数：表示设置广播模式。

