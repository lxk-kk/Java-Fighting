#### Reactor

##### 介绍

+ Reactor 反应器模式事高性能网络编程 在设计和架构层面的基础模式！

  例如：Web 服务器 Nginx、缓存中间件 Redis、通信中间件 Netty 等高性能的中间件都是基于 反应器模式 实现网络通信的。

+ 反应器模式由 Reactor 反应器 和 Handler 处理器两大角色组成。

  1. Reactor 反应器

     负责查询 IO 事件，当检测到一个 IO 事件，就将其发送给相应的 Handler 处理器去处理。

     这里的 IO 事件，就是 NIO 中选择器监控的通道 IO 事件。

  2. Handlers 处理器

     与 IO 事件（或者选择键）绑定，负责 IO 事件的处理。

     完成真正的 连接建立、通道读取、业务逻辑处理、将结果写到通道等。

##### 单线程 Reactor

+ 单线程 Reactor 反应器模式

  Reactor 反应器和所有的 Handler 处理器处于一个线程中执行，是最简单的 反应器模型。

  ![](image\单线程 Reactor.jpg)

+ 基于 NIO 的单线程 反应器模式：

  + 需要用到 SelectionKey 选择键的两个成员方法：

    1. `void attach(Object o)`

       ```
       将 任意的 Java POJO 对象，作为附件添加到 SelectionKey 实例中。
       在 单线程反应器模式中，需要将 Handler 处理器实例，作为附件添加到 SelectionKey 中。
       即：为各个 SelectionKey 绑定 Handler。
       ```

    2. `Object attachment()`

       ```
       取出通过 attach(Object o) 方法添加到 SelectionKey 选择键实例的附件。
       即：取出与 SelectionKey 绑定的 Handler
       ```

       ```
       在反应器模式中，需要将 attach 和 attachment 结合使用：
       · 在选择键注册完成之后，调用 attach 方法，将 Handler 处理器绑定到 选择键上；
       · 当事件发生时，调用 attachment 方法，可以从选择键中取出 Handler 处理器，将事件分发到 Handler 处理器中，完成业务处理。
       ```

  + 根据 业务需求定义两个 Handler：AcceptHandler 和 IOHandler

    + AcceptHandler

      最开始时，绑定 AcceptHandler 处理接受新连接事件，并将 SelectionKey 中绑定达 handler 修改为 IOHandler

      ```java
      class AcceptHandler implements Runnable{
          // serverSocket = ...
          // selector = ...
          public void run(){
              try{
                  SocketChannel channel = serverSocket.accept();
                  if(channel != null){
                      new IOHandler(selector, channel);
                  }
              }catch(IOException e){
                  log.error("handler error!");
              }
          }
      }
      ```

    + IOHandler

      处理 通道的 IO 事件，此次通信的通道只会建立一次连接，因此，可以直接使用 IOHandler 替换 AcceptHandler

      ```java
      class IOHandler implements Runnable{
          final SocketChannel channel;
          final SelectionKey selectionKey;
          IOHandler(Selector selector, SocketChannel channel){
              this.channel=channel;
              selectionKey = channel.register(selector,SelectionKey.OP_READ);
              // 将 IOHandler 绑定到 SelectionKey
              selectionKey.attach(this);
              selector.wakeup();
          }
          public void run(){
              // 1:处理 read
              // 读取完毕后，需要注册 write 就绪事件
              
              // 2:处理 write
              // 写完毕后，需要注册 read 就绪事件
          }
      }
      ```

  + Handler 的触发：

    selector 查询所有 SelectionKey 的时候，将所有就绪的 SelectionKey 放到一个 Set 中。

    处理就绪事件时，遍历 Set，取出单个 SelectionKey

    处理事件的业务逻辑时，取出 SelectionKey 中绑定的 handler，并执行其 run 方法即可。

    ```java
    while(true){
        selector.select();
        Iterator ite=selector.selectedKeys().iterator();
        while(ite.hasNext()){
            SelectionKey selectionKey=(SelectionKey)ite.next();
            Runnable handler=(Runnable)selectionKey.attachment();
            if(handler != null){
                handler.run();
            }
        }
    }
    ```

    虽然 Handler 实现了 Runnable 接口，但是并没有新建一个线程执行，因为这里演示的只是 单线程模型。

+ 单线程 Reactor 模式的缺点：

  单线程反应器模式中，Reactor反应器 和 Handler 处理器都执行在同一条线程上。

  因此，当其中某个一 Handler 阻塞时，会导致其他所有的 Handler 都得不到执行。在这种场景下，如果被阻塞的 Handler 不仅仅负责的是 输入和输出业务，还包括 负责连接监听的 AcceptorHandler 处理器，那么这个 AcceptorHandler处理器将被阻塞，如此以来，会导致整个服务不能接受到新的连接，使得服务器变得不可用。

  基于以上原因，单线程反应器模型使用的较少。

##### 多线程 Reactor

1. 将负责输入输出处理的 IOHandler 处理器的执行，放入独立的线程池中。

   使得业务处理线程 与 负责服务监听 和 IO事件查询的反应器线程相互隔离，避免服务器的连接监听受到阻塞。

2. 如果服务器为多核 CPU，可以将反应器线程拆分成多个 子反应器线程（SubReactor），相应的每个反应器都独立负责一个选择器。充分利用系统资源的同时，也使得反应器能够管理大量的连接，提升了通信的性能。

##### 总结

+ Reactor 反应器模式中 IO 事件处理流程

  <img src="image\Reactor 中 IO 事件处理.jpg" style="zoom:90%;" />

  

  1. 注册通道（IO 源于通道）

     对于底层连接而言，IO 和 通道是强相关的。一个 IO 事件一定属于某个通道。

     因此：如果要查询通道中的某个事件，首先需要将 通道注册到 选择器中，一旦某个事件到来，选择器就会从通道中查询到该事件。

  2. 查询选择

     在反应器模式中，一个反应器（或者 SubReactor 子反应器）会负责一个线程。

     通过不断轮询，查询选择器中的 IO 事件（SelectionKey）。

  3. 事件分发

     如果查询到 IO 事件，则将该事件发送给与其绑定 Handler 处理器进行处理。
     
  4. Handler 处理器处理事件
  
     整个 反应器模式的 IO 事件处理