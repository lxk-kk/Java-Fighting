### 【线程_Java】

#### 线程：生命周期 和 状态切换

+ Java 线程的 6 种状态

  ![](image/Java线程的生命周期.jpg)

  1. NEW —— 新建状态：线程被创建后，还未执行 start() 方法前的状态

     ```java
     /*
      · new Thread(); 便是新建状态
     */
     ```

  2. RUNNABLE —— 运行中状态：Java线程将操作系统中的`就绪态`和`运行态`统称为 `运行中 Runnable`

     ```java
     /*
      · 就绪态（Ready）：线程对象的 start() 方法被执行后，等待 CPU分配时间片，获取 CPU的使用权 的状态！
      · 运行态（Running）：线程获得 CPU 的使用权，执行任务的状态！
      
      · Ready -> Running ：线程得到 CPU 使用权
      · Running -> Ready ：线程执行 yield() ：主动让出 CPU 使用权，并进入就绪态！
     */
     ```

  3. BLOCKED —— 阻塞状态：线程获取锁失败后，阻塞于锁的状态；线程执行 阻塞式 IO 请求，被阻塞的状态！

     ```java
     /*
      · Synchronized 加锁失败，等待加锁 ：阻塞状态
             进入 Synchronized 方法
             进入 Synchronized 代码块
       	-> Runnable（Ready）：加锁成功！
       	
      · 阻塞式 IO 请求：线程被阻塞！
      	-> Runnable（Ready）：IO处理完毕
     */
     ```

  4. WAITING —— 等待状态：表示进程进入等待状态，需要`等待`其他线程 `通知`或者`中断` 当前线程

     ```java
     /*
      · obj.wait() ：当前线程执行 obj.wait()，则释放锁，进入等待状态！
      	-> Runnable（Ready）：其他线程执行 obj.notity() / obj.notityAll() ！
      	
      · thread.join() ：当前线程执行 thread.join()，则当前线程进入等待状态！
      	-> Runnable（Ready）：thread 线程执行完毕！
     */
     ```

  5. TIME_WAITING —— 超时等待状态：与 WAITING 状态不同。在指定的时间后，当前线程还未等到其他线程的 通知或者中断，则自行返回！

     ```java
     /*
      · obj.wait(long timeout) ：当前线程执行 obj.wait(timeout)，则释放锁，进入超时等待状态！
      	-> Runnable（Ready）：其他线程执行 obj.notity() / obj.notityAll() ，到期自动唤醒！

      · thread.join(long timeout) ：当前线程执行 thread.join(timeout)，则当前线程进入超时等待！
      	-> Runnable（Ready）：thread线程执行完毕，到期自动唤醒！

      · Thread.sleep(long timeout) ：调用 Thread.sleep(long timeout)，则 当前线程进入超时等待！
      	-> Runnable（Ready）：到期启动唤醒！
     */
     ```

  6. TERMINATED —— 终止状态：表示当前线程已经执行完毕！

     ```java
     /*
      · run() / call() 方法执行完成！
      · stop() 方法结束线程 —— 容易造成死锁，被废弃！
      · 异常抛出：线程抛出一个未捕获的异常 Exception 或者 Error：线程结束！
     */
     ```

#### Daemon 守护线程

```java
/*
 · 守护线程是程序运行时，在后台提供服务的线程，可有可无！
 · 当所有 非守护线程 结束运行时，程序就会终止，同时会杀死所有守护线程（不论其是否在执行）！
 · 执行 main() 方法的线程是主线程，不是 守护线程！
 · 典型引用：GC（执行垃圾回收器的线程）
 · thread.setDaemon( true )：设置 thread 为守护线程！
*/

// 示例
private static class ThreadTest implements Runnable{
    @Override
    public void run(){
        try{
            Thread.sleep(100L);
        }catch(InterruptedException e){
            // ignore
        }finally{
            System.out.println("execute deamon thread");
        }
    }
}
public static void main(String ...args){
    ThreadTest task = new ThreadTest();
    Thread thread = new Thread(task);
    thread.setDeamon(true);
    thread.start();
}
// 将不会输出 execute deamon thread !
// 主线程 启动完 thread线程 之后，程序便结束，thread线程（deamon 线程）也被杀死！
```

#### 线程：创建、启动、终止

##### 线程创建：4 种方式

```java
/*
 1、继承 Thread 类，重写 run() 方法
 2、实现 Runnable 接口，实现 run() 方法
 3、实现 Calllable 接口，实现 call() 方法
 4、使用 Executor 框架创建线程池
*/
```

###### 1、继承 Thread 类

```java
class ThreadTest extends Thread{
    @Override
    public void run(){
        // your service codes
    } 
}

// 创建
ThreadTest thread=new ThreadTest();
// 启动
thread.start();
```

###### 2、实现 Runnable 接口

```java
class ThreadTest implements Runnable{
    @Override
    public void run(){
        // your service codes
    }
}

// 创建
ThreadTest task=new ThreadTest();
Thread thread=new Thread(task);
// 启动
thread.start();
```

###### 3、实现 Callable 接口

```java
class ThreadTest implements Callable<String>{
    // Callable call 方法会抛出异常！
    @Override
    public String call() throws Exception  {
        // your service codes
        return "your expected result";
    }
}
// 创建
ThreadTest task = new ThreadTest();
FutureTask<String> futureResult = new FutureTask<>( task );
Thread thread = new Thread( futureResult );
// 启动
thread.start();
// 获取结果（阻塞）
String result = futureResult.get();
```

###### 4、Executor 框架创建线程池

```java
// ThreadPoolExecutor 构造器创建 ———— 推荐
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    10,// corePoolSize
    20, // maximumPoolSize
    60L, // keepAliveTime
    TimeUnit.SECONDS, // TimeUnit
    new ArrayBlockingQueue<Runnable>(50) // workQueue
);

// Executors 工具类创建 ———— 不推荐
ExecutorService service1 = Executors.newFixedThreadPool( nThreads ); // LinkedBlockingQueue
ExecutorService service2 = Executors.newSingleThreadExecutor(); // LinkedBlockingQueue
ExecutorService service = Executors.newCachedThreadPool(); // maximumPoolSize
ExecutorService service3 = Executors.newScheduledThreadPool( corePoolSize); // maximumPoolSize
```

##### 线程启动：start() VS run()

```java
/*
面试题：为什么不直接调用 Thread 的  run() 方法启动线程，而是调用 start() 方法

 · new Thread() 会创建一个线程，该线程处于新建状态！
 
 · start
 	start 方法的含义：当前线程（即parent线程）同步告知 java 虚拟机只要线程规划器空闲，应立即启动调用 start 方法的线程，线程启动后，便进入就绪状态，此时线程才能被 JVM 调度执行！若线程获得 CPU 执行权，则线程进入运行状态，并自动调用 run 方法执行线程的主体任务！
 	
 · run
 	run 方法是个普通方法，提供给开发者定义线程执行的主体任务，只有当线程启动，并获取 CPU 执行权时，才会执行 run 方法的内容！
   若直接调用 run() 方法，则该方法的任务将会被调用者线程执行，而并不会启动一个新的线程，线程的启动，只能通过 start() 方法！
*/
```

##### 线程中断

###### 中断机制

```java
/*
 · 中断机制：
 	Java 中断机制是一种协议机制：因为它并不能直接中止一个线程，它只是改变该线程的中断状态！具体是否需要中断线程，得判断线程状态，若线程处于 BLOCKED/WAITING/TIMED_WAITING 状态，则线程是否在中断，还是得看线程对中断异常的处理！（需要注意：中断机制 并不会中断 IO阻塞 和 synchronized阻塞）
 
 · 中断原理：
	1、若线程处于 运行态/IO阻塞/synchronized阻塞，则它不会查看中断标识，尽管中断标识为 ture，依旧不影响！
	2、如果线程处于 BLOCKED/WAITING/TIMED_WAITING 状态，则该线程会一直查看自己的中断标识（默认为 false：不中断），如果检测到中断标识为 true，则线程中断阻塞，并抛出异常 InterruptException：这是个 受查异常，需要在程序中提供异常处理器！

 · Java 中断模型
	每个线程对象都有自己的中断标识(boolean)，该标识位于 虚拟机层面，在jdk源码中不可见！它代表着是否有中断请求
（该请求可以来自所有线程，可以来自线程本身）
*/


/**
 * 自行决定是中断线程！
 * @author 10652
 */
public class ThreadTest {

    public static void main(String[] args) {
        ThreadRunnable1 task1 = new ThreadRunnable1();
        ThreadRunnable2 task2 = new ThreadRunnable2();
        Thread thread1 = new Thread(task1);
        Thread thread2 = new Thread(task2);
        thread1.start();
        thread2.start();
        // 中断标识
        thread1.interrupt();
        // 中断标识
        thread2.interrupt();
        System.out.println("main thread running");
    }
    private static class ThreadRunnable1 implements Runnable {
        @Override
        public void run() {
            long threadId = Thread.currentThread().getId();
            System.out.println("current thread " + threadId + " start!");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                System.out.println("current thread " + threadId + " is interrupted!");
            }
            System.out.println("current thread " + threadId + " end!");

        }
    }
    private static class ThreadRunnable2 implements Runnable {
        @Override
        public void run() {
            long threadId = Thread.currentThread().getId();
            System.out.println("current thread " + threadId + " start!");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                System.out.println("current thread " + threadId + " is interrupted!");
                throw new RuntimeException( threadId + " the sleep thread is interrupted");
            }
            System.out.println("current thread " + threadId + " end!");

        }
    }
}
// 执行结果如下：
// thread 13 中断捕获后，继续执行；
// thread 14 中断捕获后，继续抛出异常，停止执行！
main thread running
current thread 13 start!
current thread 13 is interrupted!
current thread 13 end!
current thread 14 start!
current thread 14 is interrupted!
Exception in thread "Thread-1" java.lang.RuntimeException: 14 the sleep thread is interrupted
	at synchronizedtest.ThreadTest$ThreadRunnable2.run(ThreadTest.java:46)
	at java.base/java.lang.Thread.run(Thread.java:844)
```

###### 中断相关方法

1. Thread 类中的实例方法：void interrupt()

   ```java
   /*
    · 中断线程方法：由线程实例调用，并被本线程或者其他线程执行！当本线程被阻塞以后，其他线程执行，便会修改被阻塞线程的中断状态！（如何调用：被阻塞线程实例.interrupt()）
   */
   ```

2. Thread 类中的实例方法: boolean isInterrupted()

   ```java
   /*
    · 测试中断状态：由线程实例调用，若线程的中断状态为true，则返回true！
   */
   ```

3. Thread 类中的静态方法：static boolean interrupted()

   ```java
   /*
    · 测试当前线程的中断状态，返回当前线程的中断状态，并清空中断状态（设置为 false）
    
    · 该类方法应用广泛！
    	从 Java API中可看出，许多声明抛出 InterruptedException 的方法，都会先调用 Thread.interruped()，判断当前线程是否需要被中断，如果中断标识被设置为 ture，则将中断标志位重置，并抛出异常！
    	例如：reentrantLock.lockInterruptibly()、reentrantLock.await()等方法！
    	
    · 注意：Thread.sleep(long)、thread.join()、obj.wait() 等也会抛出 InterruptedException 异常，并且会重置中断标识，它们都是通过本地方法实现的！
   */
   ```

###### 如何处理中断

```java
/*
 · 对于 java 线程来说，调用一个阻塞方法（wait()、sleep()、join() 等），需要处理这些方法中抛出的受查异常 InterruptException！有以下两种处理方式：
    1、将中断异常继续抛出，在方法声明中 InterruptException，将 中断异常交由上层调用者处理，本线程便终止执行！
    2、内部处理 中断异常，本线程可以继续执行！捕获 InterruptException 异常后，需要及时清除 Interrupt 状态！
*/
```

##### 被废弃的方法

```java
/*
 · stop() ：public final void stop() ：实例方法，天生不安全！可被本线程的终止，也可被其他线程终止！
 	终止一个线程，同时终止线程上所有方法，包括 run() 方法！当线程被终止，立即释放被他锁住的所有对象的锁，这可能会使线程资源的不正常释放，导致资源的状态不一致！(如下例)
*/

//例如：转账业务 A->B
@Transcational(rollback = "Throwable.class")
public void writeData(Integer money){
    reduceBankA(money);
    // 此处 当前线程被 stop() ：A 转出了，但是 B 收不到！(线程死亡了，添加了事务也没用！)
    increaseBankB(money);
}
/*
 · suspend() ：public final void suspend() ：实例方法，容易产生死锁！
	挂起一个线程 A，方法调用后，线程 A 不会释放已经占有的资源（比如锁），而是占有着资源进入睡眠状态！在线程 A 未被恢复之前，其持有的资源是不可用，若此时，线程 B 正要获取该资源，则会被阻塞！程序死锁：A 等待被恢复，B 等待获取资源！
	（这种死锁在 jstack、jConsole 中检测不出来，只会显示线程的状态、线程是否被阻塞、是否在等待！）

 · resume() ：public final void resume() ：恢复被 suspend() 挂起的线程！
*/


/**
 * suspend() 方法产生死锁！
 * @author 10652
 */
public class ThreadTest {
    // 资源
    private static Object resource = new Object();

    public static void main(String[] args) {
        Thread threadA = new Thread(new ThreadRunnableA());
        Thread threadB = new Thread(new ThreadRunnableB(threadA));
        threadA.start();
        threadB.start();
    }


    private static class ThreadRunnableA implements Runnable {

        @Override
        public void run() {
            long threadId = Thread.currentThread().getId();
            
            synchronized (resource) {
                System.out.println("thread A " + threadId + " get resource!");
                try {
                    // 获取资源后，睡眠 100ms 模拟耗时操作！
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    System.out.println("thread A " + threadId + " is interrupted!");
                }
                System.out.println("thread A " + threadId + " end!");
            }
        }
    }

    private static class ThreadRunnableB implements Runnable {
        Thread threadA;
        
        ThreadRunnableB(Thread threadA) {
            this.threadA = threadA;
        }

        @Override
        public void run() {
            long threadId = Thread.currentThread().getId();
            System.out.println("thread B " + threadId + " start");

            // 线程 B 睡眠 50ms，等待 线程 A获取资源
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                System.out.println("thread B " + threadId + " is interrupted!");
            }
            // 线程 B 挂起线程 A
            threadA.suspend();
            // 线程 B 尝试获取 线程 A的资源
            synchronized (resource) {
                System.out.println("thread B " + threadId + " get resource!");
            }
            System.out.println("thread B " + threadId + " end!");
        }
    }
}
// 输出结果：线程 A 被挂起，线程 B 获取锁被阻塞！
thread A 13 get resource!
thread B 14 start
```

##### 终止线程的方法

```java
/*
【安全的终止】

 · 利用中断
 	中断利用线程上的标识位，简便的进行线程间的交互操作，这种方式最适合用于取消或者停止任务！
 	中断方法：thread.interrupt() thread.isInterrupted() Thread.interrupted() 
 
 · 可以设置自定义标志位（例如 boolean 类型变量），控制是否需要停止任务并终止该线程！
 
【 其他终止 】
 
 · ExecutorService.shutdown、ExecutorService.shutdownNow
 	shutdown()：修改线程池的状态位 SHUTDOWN：会等待工作队列里的任务都执行完毕后，再关闭所有线程（安全）
 	shutdownNow()：修改线程池状态位 STOP：关闭所有线程，废弃所有任务
 	
 · future.cancel
 	通过 submit 提交的任务，将会返回一个 Future 实例，通过该实例，可以取消正在异步执行的线程！
 	
【 被废弃的终止 】

 · thread.stop()：简单粗暴，不安全！
 
*/

// 两种安全的中断示例：

/**
 *
 * @author 10652
 */
public class ThreadTest {

    public static void main(String[] args) {
        SafeShutDownThread taskA = new SafeShutDownThread();
        Thread threadA = new Thread(taskA);
        threadA.start();
        try {
            // 模拟耗时动作：让主线程有机会中断线程 A
            TimeUnit.MILLISECONDS.sleep(50);
        } catch (InterruptedException e) {
        } finally {
            // 通过线程上的中断标志位 安全终止线程 A
            threadA.interrupt();
        }

        SafeShutDownThread taskB = new SafeShutDownThread();
        Thread threadB = new Thread(taskB);
        threadB.start();
        try {
            // 模拟耗时动作：让主线程有机会中断线程 B
            TimeUnit.MILLISECONDS.sleep(50);
        } catch (InterruptedException e) {
        } finally {
            // 通过自定义标识位 安全终止线程 B
            taskB.shutdown();

        }
    }

    private static class SafeShutDownThread implements Runnable {

        // 自定义关闭线程的标识！
        boolean shutdown = false;

        @Override
        public void run() {
            long threadId = Thread.currentThread().getId();
            int i = 0;
            System.out.println("thread " + threadId + " start");
            while (!shutdown && !Thread.interrupted()) {
                i++;
            }
            System.out.println("thread " + threadId + " is interrupted, the value is " + i);
        }

        // 确认关闭线程！
        void shutdown() {
            shutdown = true;
        }
    }
}
```

#### 线程：通信、协作

######  wait() / notify() / notifyAll()

```java
/*
 · wait() / notify() / notifyAll() 都是 Object 类中的 本地、实例方法！
 	- wait()			 ：public final void wait() throws InterruptedException（调用 wait(0); 实现）
 	- wait(long timeout) ：public final native void wait(long timeout) throws InterruptedException;
 	- notify()			 ：public final native void notify();
 	- notifyAll()		 ：public final native void notifyAll();
 	
 · 等待
	wait()：将当前线程挂起，线程进入 WAITING 状态；
		直到其他线程调用 notify 、notifyAll 才可能将其唤醒！
		
	wait(long timeout)：设置挂起时长，线程进入 TIMED_WAITING；
		在规定时限内，若没有其他线程 notify、notifyAll 将其唤醒，则自动唤醒！

 · 唤醒
	notify()：随机唤醒 等待队列 中的任意一个线程
	nofityAll()：唤醒 等待队列 中的所有线程
	
 · 都属于 Object 的实例方法，因为都需要依赖对象锁，进行线程之间的通信！
*/
```

`等待/通知 机制`

```java
/*
 · 等待通知机制
 	wait/notify、notifyAll 构成等待通知机制！
	它是指一个线程 A 调用了对象 obj 的 wait 方法后进入等待状态！而另一个线程 B 调用用了对象 obj 的 notify、notifyAll 方法进行通知；线程 A 收到通知后，从对象 obj 的 wait() 方法返回，进行后续操作！

 · 等待通知机制 原理浅析
	1、等待机制：在同步机制下，线程获取对象监视器，当对象调用 wait() 方法时，该线程被 对象监视器 挂起，放入对象的 [ 等待队列( WaitQueue ) ] 中，等待被唤醒。线程状态由 运行态(Running) 转变为 等待(Waiting) 状态，同时线程释放对象锁，将锁让给随后的线程（注意：此时线程已经进入对象的等待队列中）。
	
 	2、通知机制：在同步机制下，线程获取对象监视器，当对象调用 notify()、notifyAll() 方法时，该线程通过对象监视器，将处于对象的等待队列中的等待线程唤醒（notify：随机唤醒一个线程，notifyAll：唤醒等待队列中的所有线程），被唤醒的线程 将被对象监视器 放入对象的 [ 锁池：同步队列( SynchronizedQueue ) ] 中，同时线程状态变为 阻塞态(Blocked) ，在 阻塞队列中的线程 将会在 notify线程 释放锁以后 竞争锁！

 · 问：
 	为什么 wait/notify 要和 synchronized 联合使用？
 	为什么 wait/notify 是 Object 中的实例方法，而不是 Thread 类中的？
 
 · 解答：
 	由上述可知：等待通知机制 依赖于 对象obj！线程之间要想依赖于 object 进行通信，那就需要利用同步机制，获取对象锁！
 	获取对象锁 其本质就是 对象监视器（ObjectMonitor）的获取！这个获取过程是排他的，也就是同一个时刻只能有一个线程获取到由 synchronized 所保护对象的监视器！
	这就是为什么，wait、notify 需要和 synchronized 联合使用，同时也正是因为需要对象的支持，所以 wait、notify、notifyAll 方法都是 Object 类中的实例方法，并由对象调用！若不在同步块内使用，将抛出异常：IllegalMonitorStateException！
*/
```

######  sleep(timeout) / yield() / join()

```java
/*
 · sleep(timeout) / yield() / join() 都是 Thread 类中的方法！
 	- sleep ：静态本地方法 ：public static native void sleep(long millis) throws InterruptedException
	- yield ：静态本地方法 ：public static native void yield()
	- join  ：实例方法 	  ：public final void join() throws InterruptedException
 
 · sleep(long timeout) 
 	Thread.sleep(timeout)：当前线程睡眠指定的时长 —— 进入 超时等待(Time_waiting) 状态！
 	该方法不用依托于同步，可以直接由 Thread 调用，当在同步代码块中调用 sleep 方法，当前线程运行将被暂停，并且线程让出 cpu 的执行权给其他任务！而线程只是让出 cpu ，并不会释放对象上的锁！同步代码块外的线程依旧会被阻塞，拿不到锁！
	
 · yield()：
 	Thread.yield()：线程让步 —— 进入 就绪态（Ready）
	暂停当前线程，以便其他线程有机会执行，yield方法将 Running 状态转变为 Runnable 状态！很少有场景会用到该方法，主要使用的地方是调试、测试并发程序！
	
 · join() / join(long timeout)
 	thread.join() / thread.join(timeout)：当前线程等待其他线程终止 —— 进入 WAITING/TIMED_WAITING 状态！
	在线程 A 中执行 线程 B 的 join 方法时（即：threadB.join()），线程 A 被阻塞，直到线程 B 执行完成！此时，线程 A 由等待状态 变为 就绪状态！ 
	使用场景：父线程等待子线程执行完成以后再执行，将异步的父子线程转换为同步线程！例如：主线程生成并启动了子线程，当主线程需要用到子线程的返回结果时（主线程需要等到子线程执行结束后才能结束），子线程可调用 join() 方法！
*/


/**
 * join() 方法使用示例
 * @author 10652
 */
public class ThreadTest {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + " start");
        Thread thread = new Thread(new JoinTest());
        thread.start();
        try {
            // 主线程 执行 线程thread 的 join 方法：主线程被阻塞
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 直到 thread 线程执行完，才会执行！
        System.out.println(Thread.currentThread().getName() + " end");
    }

    private static class JoinTest implements Runnable {
        @Override
        public void run() {
            System.out.println("thread " + Thread.currentThread().getName() + " start");
            try {
                // 模拟耗时操作
                TimeUnit.MILLISECONDS.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("thread " + Thread.currentThread().getName() + " end");
        }
    }
}
// 执行结果
main start
thread Thread-0 start
thread Thread-0 end
main end
```

######  wait() VS sleep()

```java
/*
【 相同点 】
	1、wait()、sleep() 方法都是将线程挂起，使得当前线程在一段时间内不能执行！
	2、挂起状态都是可中断的：其他线程调用 interrupt() 方法都会抛出 InterruptedException 异常，挂起状态结束！
	3、都可设置超时时间！

【 不同点 】
   1、wait() 需要对象的支持，所以 wait() 是 Object 的实例方法；sleep() 针对于当前线程操作，与对象无关，所以是 Thread 的静态方法
   2、wait() 需要借助 对象监视器 ，所以只能在同步代码块中使用，否则抛出 IllegalMonitorStateException；sleep() 任意地方都可使用
   
   3、wait() 执行后会释放对象锁，其余线程竞争对象锁；若在同步代码块中执行 sleep() 则当前线程只会睡眠，让出 cpu 的执行权，不会释放锁，其余线程依旧阻塞等待锁被释放！
   
   4、wait() 被挂起的线程，需要由其他线程唤醒，也可以设置超时自动唤醒，唤醒后需要和其他线程竞争锁，只能竞争成功后才能执同步代码块；sleep() 挂起的线程，在指定的时间过后在自动醒来，并接着执行！

   5、wait()/notify() 体现的是不同线程之间的协作通信；sleep() 就是当前线程的调度，不涉及其他线程！

*/
```

###### await() / signal() / signalAll()

```java
/*
 · java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其他线程调用 signal() 或者 signalAl() 方法唤醒等待的线程！ 
 · 使用 Lock 来获取一个 Condition 对象（ Lock是所有锁的接口 ）

 · 【 注意 】
 	Object 的 wait() / notify() 与 同步机制synchronized（对象监视器） 搭配使用完成 等待通知机制 —— 实现在 JVM 层面的！
 	Condition 的 await() / signal() 与 Lock 搭配使用完成 等待通知机制 —— 实现在 java 语言层面！（同样释放锁 lock）

 · 【 区别 】
	1、Condition 可以支持不响应中断，Object 的 wait 必须响应中断！
	2、Condition 能够支持多个等待队列（一个 lock 可以 创建多个 condition对象），而 Object 只能支持一个。


 · 等待：对应 obj.wait() / obj.wait(long timeout)
	condition.await() ：当前线程进入等待状态，释放 lock 锁！如果等待状态被中断，则抛出中断异常！
	condition.await(timeout) ：超时等待：等待其他线程调用 condition.signal() / condition.signalAll()，超时自动唤醒！响应中断！
	condition.awaitUninterruptibly() ：不响应中断！
 
 · 通知：对应 obj.notify() / obj.notigyAll()
 	condition.signal()	：唤醒一个等待在 condition 等待队列 中的线程，线程被移置 同步队列 中，可能与其他线程竞争 lock
 	condition.signalAll() ：唤醒 condition 等待队列上的所有线程
*/


/**
 * 使用示例：
 * @author 10652
 */
public class ThreadTest {

    private ReentrantLock reentrantLock = new ReentrantLock();
    private Condition condition = reentrantLock.newCondition();

    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        ThreadTest threadTest = new ThreadTest();
        service.execute(() -> threadTest.after());
        service.execute(() -> threadTest.before());
    }

    public void before() {
        reentrantLock.lock();
        try {
            System.out.println("before " + Thread.currentThread().getId() + " before");
            condition.signalAll();
            System.out.println(Thread.currentThread().getId() + " after");
        } finally {
            System.out.println(Thread.currentThread().getId() + " unlock");
            reentrantLock.unlock();
        }
    }

    public void after() {
        reentrantLock.lock();
        try {
            System.out.println("after " + Thread.currentThread().getId() + " before");
            condition.await();
            System.out.println(Thread.currentThread().getId() + " after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getId() + " unlock");
            reentrantLock.unlock();
        }
    }
}

```

+ [Condition 用法](https://www.cnblogs.com/jalja/p/5895051.html)

+ [Condition await / signal 原理](https://www.jianshu.com/p/28387056eeb4)

- [LockSupport 用法及原理](https://www.jianshu.com/p/f1f2cd289205)