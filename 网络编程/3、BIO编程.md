#### BIO编程

##### 介绍

+ Blocking IO 是**同步阻塞**编程方式（是最原始的 IO编程方式），通常是在 JDK1.4版本之前常用的编程方式。


##### 实现

1. 首先在服务端启动一个 ServerSocket ，由 acceptor 监听网络请求
2. 客户端启动 Socket 发起网络请求，请求 socket连接
3. 服务端监听到客户端的 网络请求，默认情况下 ServerSocket 会建立一个与客户端平等的 socket ，并创建一个新线程处理该请求，如果服务端没有线程可用，客户端则会阻塞等待或遭到拒绝。
4. 建立好 socket连接 之后，服务端的 socket 和客户端的 socket 将会直接进行网络通讯，双方均可进行 IO操作
5. 当有其他 客户端发起网络请求时，服务端同样会建立 一个新的 socket 处理该客户端的请求，并进行通讯！

+ **注意：**

  acceptor 始终监听的端口号！当IO结束，任何一方结束socket 连接，则整个socket 线程结束！建立的socket通信是长连接，连接完成后可以多次请求响应。

  ```
  短连接，如HTTP 1.0 协议，就是一次连接只供一个请求和应答！
  ```

+ 建立好的连接，在通讯过程中是同步的，在并发的处理效率上比较低，大致结构如下所示：

  <img src="image\BIO 通信.png" alt="BIO 通信" style="zoom: 67%;" />

+ 注意

  1. 同步阻塞 IO：服务器实现模式为**一个连接对应一个线程**，即客户端有连接请求时服务端就需要启动一个线程进行处理，如果这个连接不做任何事就会造成不必要的线程开销！

  2. **启用线程池机制**：由线程池管理 socket线程 的连接处理

     有人称线程池机制为**伪异步**，但实际上使用线程池机制并不能实现异步操作，它只是线程池用来处理优化服务器的一种方式！

  3. **BIO适用于连接数目较小，并且大小固定的架构，这种方式对服务器资源要求比较高！**并发局限于应用中，是JDK1.4 以前的唯一选择。

##### 代码示例

+ *Server端*

  ```java
/**
   * @author 10652
   */
  public class Server {
      public static void main(String[] args) {
          int port = genPort(args);
          ServerSocket serverSocket = null;
          try {
              // 启动 ServerSocket
              serverSocket = new ServerSocket(port);
              System.out.println("Server : server started");
              while (true) {
                  // 阻塞 监听
                  Socket socket = serverSocket.accept();
                  // 当有客户端连接，启动一个线程进行处理
                  new Thread(new Handler(socket)).start();
              }
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              if (!Objects.isNull(serverSocket)) {
                  try {
                      serverSocket.close();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  
      public static int genPort(String... args) {
          try {
              if (args.length > 0) {
                  return Integer.parseInt(args[0]);
              }
          } catch (NumberFormatException n) {
              return 9999;
          }
          return 9999;
      }
  
      private static class Handler implements Runnable {
          Socket socket;
          Handler(Socket socket) {
              this.socket = socket;
          }
  
          @Override
          public void run() {
              BufferedReader reader = null;
              PrintWriter writer = null;
              try {
                  reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                  writer = new PrintWriter(new ObjectOutputStream(socket.getOutputStream()));
                  String readMsg = null;
                  while (true) {
                      System.out.println("server reading...");
                      // 服务端接收信息：readLine
                      if ((readMsg = reader.readLine()) == null) {
                          break;
                      }
                      // 服务端打印 接收到的信息
                      System.out.println("Server ：接收到消息：" + readMsg);
                      // 服务端通知客户端，已经获取到消息
                      writer.println("server recive : " + readMsg);
                      // 刷新缓冲区
                      writer.flush();
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  
  ```
  
+ *Client端*

  ```java
  /**
   * @author 10652
   */
  public class Client {
      public static void main(String[] args) {
          // 设定要连接的服务端的ip和port
          String host = null;
          int port = 0;
          if (args.length > 2) {
              host = args[0];
              port = Integer.parseInt(args[1]);
          } else {
              host = "localhost";
              port = 9999;
          }
          Socket socket = null;
          BufferedReader reader = null;
          PrintWriter writer = null;
  
          Scanner in = new Scanner(System.in);
          try {
              // 客户端 启动Socket：与服务端建立连接请求（host,port是服务端）
              socket = new Socket(host, port);
              String msg = null;
              reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), "utf-8"));
              writer = new PrintWriter(socket.getOutputStream(), true);
              while (true) {
                  // 键盘输入
                  msg = in.nextLine();
                  if (Objects.equals(msg, "exit")) {
                      break;
                  }
                  // 将输入内容传递给 服务端
                  writer.println(msg);
                  // 强制刷新缓冲区
                  writer.flush();
                  // 接收服务端回传信息
                  System.out.println("Client：接收 Server 端回传消息：" + reader.readLine());
              }
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              if (!Objects.isNull(socket)) {
                  try {
                      socket.close();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  }
  
  ```

##### 优化

+ 使用线程池机制改善后的 BIO 模型

  <img src="image\线程池机制改善 BIO 模型.png" style="zoom:67%;" />

  + 服务器端依旧通过 acceptor 监听网络请求，客户端发送 socket请求，服务端创建一个 socket与之连接，并将该socket请求任务放入线程池中，交由线程池管理执行任务，线程池与客户端进行一一的对应。

+ 代码示例

  ```java
  /*
  服务端代码仅仅将创建创建线程提交任务更换为将任务交由线程池管理！
  客户端代码一点不变！
  */
  【Server端】
  public static void main(String[] args) {
      int port = genPort(args);
      ServerSocket serverSocket = null;
      // ---------变化1：创建线程池
      ExecutorService pool = Executors.newFixedThreadPool(50);
  
      try {
          serverSocket = new ServerSocket(port);
          System.out.println("Server : server started");
          while (true) {
              Socket socket = serverSocket.accept();
              // ---------变化2：将任务交由线程池处理
              pool.execute(new Handler(socket));
          }
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          if (!Objects.isNull(serverSocket)) {
              try {
                  serverSocket.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```

  