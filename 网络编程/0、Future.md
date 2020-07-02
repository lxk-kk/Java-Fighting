##### Future

+ 异步回调

  微服务架构中，频繁需要跨设备服务的接口调用，这里涉及到线程的异步回调问题。

  例如：某个业务流程，需要 N 次调用第三方接口，获取 N 种上游数据，此时便需要高效的异步调用第三方接口，并同步处理这些接口的返回接口。

  在 Netty 源代码中，大量地使用了这种异步回调技术，并且基于 Java 的异步回调，设计了自己的一整套异步回调接口和实现。

##### join 异步阻塞

+ Java 线程可以通过 join 方法，实现线程的异步阻塞。

  如果 线程A 调用了线程 B 的 B.join 方法，合并 B 线程，那么A进入阻塞状态，直到 B 线程执行完成。

+ join 方法是 Thread 类的实例方法，并不是 静态方法，需要使用线程实例调用。

  join 方法支持超时等待。

+ 缺点：

  join 方法不支持返回值，如果从异步任务中获取返回值，则可以使用 FutureTask 类。

##### Future 异步回调

+ Future + Callable 实现异步回调

+ FutureTask 代表一个未来执行的任务，它实现了 Future 接口，提供了外部操作异步任务的能力。

   FutureTask 表示新线程所执行的操作，位于 juc 包中。

+ FutureTask 内部有一个 Callable 类型的成员，代表异步执行的 逻辑。

  此外，内部还有个一 Object 类型的成员，用户保存异步保存的结果！

##### Netty 异步回调

+ Netty 官方文档中指出 Netty 的网络操作都是 异步的，在 Netty 源码中大量使用了异步回调处理模式。在 Netty 业务开发层面，Netty 应用的 Handler 处理器中，业务处理程序也是异步执行的。

+ Netty 实现了自己的异步回调体系：

  它继承和扩展了 JDK Future 系列异步回调的 API，定义了自身的 Future 系列接口和类，实现了异步任务的监控、异步执行结果的获取。

+ Netty 异步任务的扩展：

  1. 继承 Jdk 中的 Future 接口，得到属于 Netty 的 Future 异步任务接口。

     ```
      · Netty 的 Future 接口对原来的 Future 接口继续了增强，能够对执行过程进行监控，对异步回调完成事件进行监听。
      
      · Listener 接口一般不会直接使用，而是会使用 子接口，每个子接口都代表不同的异步任务。
      	例如：ChannelFuture 接口
     ```

     `Future`

     ```java
     public interface Future<V> extends java.util.concurrent.Future<V>{
         // 判断异步执行是否成功
         boolean isSuccess();
         
         // 判断异步执行是否取消
         boolean isCancellable();
         
         // 获取异步任务异常的原因
         Throwable cause();
         
         // 增加异步任务执行完成与否的监听器 Listener
         Future<V> addListener(GeneriaFutureListener<? extends Future<? super V>> listener);
         
         // 移除异步任务执行完成与否的监听器 Listener
         Future<V> removeListener(GenericFutureListener<? extends Future<? super V>> listener);
         
         // ...
     }
     ```

  2. 引入新接口：`GenericFutureListener`

     ```
      · GenericFutureListener 接口用于封装 异步非阻塞回调 的逻辑，是异步执行完成事件的 监听器。
      	Netty 使用了监听器模式，异步任务执行完成后的回调逻辑抽象成了 Listener 监听器接口。
      	
      · 可以将 GenericFutureListener 监听器接口加入 Netty 异步任务 Future 中，实现对 异步任务 执行状态的事件监听。
      
       · GenericFutureListener 接口在 Netty 中是一个基础类型接口，在网络编程的异步回调中，一般使用 Netty 中提供的某个子接口，例如：ChannelFutureListener
     ```
     
     `GenericFutureListener`
     
      ```java
     package io.netty.util.concurrent;
     import java.util.EventListener;
     
     // Future 接口是 Netty 自定义的 Future？
     public interface GenericFutureListener<F extends Future<?>> extends EventListener{
         // 监听器的回调方法：表示异步任务操作完成。在 Future 异步任务执行完成之后，将回调此方法！
         // 多数情况下，Netty 的异步回调的代码编写在 GenericFutureListener 实现类的 operationComplete 方法中。
         void operationComplete(F var1) throws Exception;
     }
     
     // EventListener 接口是一个空接口，仅仅具有标志作用，没有任何抽象方法。
      ```

+ **ChannelFuture**

  + 在 Netty 网络编程中，网络连接通道的输入和输出处理都是异步进行的，都会返回一个 ChannelFuture 接口的实例，作为`异步任务实例`。
  
    可以为 异步任务实例 添加异步回调的监听器，在异步任务真正完成之后，回调才会执行。
  
  + 示例：`ChannelFuture + ChannelFutureListener` 实现异步的 connection
  
    ```java
    // connect 是异步的，仅提交异步任务
    ChannelFuture future = bootstrap.connect( new InetSocketAddress("www.manning.com", 80));
    // connect 的异步任务真正执行完成后，future 回调监听器才会执行
    future.addListener(new ChannelFutureListener(){
        @Override
        public void operationComplete(ChannelFuture channelFuture) throws Exception{
            if(channelFuture.isSuccess()){
                System.out.println("Connection established");
            }else{
                Systeml.err.println("Connection attempt failed");
                channelFuture.cause().printStackTrace();
            }
        }
        
    });
    ```

##### Netty 出入站异步回调

+ Netty 的**出站**和**入站**都是异步的。异步回调的方法，和 Netty 建立异步回调是一样的。

  ```
  入站：输入
  出站：输出
  ```

+ ChannelInboundHandler：处理入站的 IO 事件的接口

  ChannelInboundHandlerAdapter：提供的入站处理的默认实现。

  如果想要实现自己的入韩处理器 Handler，只要继承 ChannelInboundHandlerAdapter 入站处理器，再写入自己的入站处理的业务逻辑。在 channelRead 方法中可以读取入站的数据！

+ 示例：NIO 出站 --- write
  
  + 在调用 write 操作之后，Netty 并没有完成对 Java NIO 底层连接的写入操作，因为是**异步的：NIO 同步非阻塞啊！**
  
  ```java
  // write 输出方法：返回的是一个异步任务
  ChannelFuture future = ctx.channel().write(msg);
  
  // 为异步任务，添加上监听器
  future.addListener(new ChannelFutureListener(){
      @Override
      public void operationComplete(ChannelFuture future){
          // write 操作完成之后的回调
      }
  })
  ```
  
  + 在调用 write 操作之后，立即返回一个 ChannelFuture 接口的实例。通过这个实例，可以绑定异步回调监听器，这里的异步回调逻辑需要我们编写。