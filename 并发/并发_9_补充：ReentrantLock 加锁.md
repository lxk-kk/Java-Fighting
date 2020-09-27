#### ReentrantLock

+ ReentrantLock 是 java.util.concurrent 包中的锁！**它继承于 locks 包下的 Lock 类，是 java 语言层面上的悲观锁，也是可重入锁，它实现的功能是 synchronized关键字的超集！是java语言实现的互斥同步的手段！**

  **ReentrantLock 在内部基于 AQS 框架，实现了独占锁 Sync，并在 Sync 基础上，定义了公平锁类与不公平锁类，因此 ReentrantLock 可以指定实现公平/不公平锁，默认不公平！**

  ```
   · AQS（AbstractQueuedSynchronizer）是一个抽象类，用于实现 同步器组件！AQS 借助模板方法模式，让 ReentrantLock 实现其抽象方法 tryAcquire 以及 tryRelease 以实现自定义的锁分配！而AQS 底层维护了 volatile 修饰的整型变量 state 表示锁状态，并通过 CAS 修改该状态实现同步机制！此外，AQS 还维护了一个 双端队列，用于存放需要阻塞的线程！
  ```

+ JDK 1.6 之前，Synchronized 是完全的重量级锁，*一言不合就加锁，一言不和就阻塞，这些都需要内核态的支持*，都是耗时的过程！因此，ReentrantLock 不仅能够实现 更多的功能，性能上还比 Synchronized 优秀！

  ```
  1. ReentrantLock 并不是通过操作系统的的 MUTEX 实现的互斥锁！
  	ReentrantLock 是基于 AQS 中的 锁状态实现的 资源互斥，同时 记录当前独占锁的拥有者！
  	锁状态：volatile 修饰的 整型变量，使用 AQS 的方式修改锁状态！
  	加锁解锁的方式 本身就比 Synchronized 轻量级！

  2. ReentrantLock 虽然也可以阻塞线程，但是并不是立刻阻塞！
  	首先会尝试通过 CAS 获取锁，失败之后，在真正阻塞线程之前，至少会做一次自旋，也就是说，阻塞之前会 多次尝试获取锁的操作！
  	这就使得，如果 锁冲突不大，并且持有锁的线程能马上执行完成时，线程不会盲目的 被阻塞！
  	尽管存在并发冲突，如果冲突不严重，能够避免 线程被频繁阻塞唤醒的 操作，降低资源的消耗！
  	
  3. ReentrantLock 提供了更多的 加锁方式：
  	加锁失败，则返回 —— 不会阻塞线程！
  	加锁失败，则阻塞，响应中断 —— 可以在线程阻塞后，通过中断标志位 唤醒线程！
  	加锁失败，则阻塞，超时等待，响应中断 —— 阻塞线程之前，会进行一系列的优化操作，并且阻塞线程后，超时自动唤醒！
  ```

  *JDK1. 6 开始，Synchronized 做了一些列的优化，到目前而言，ReentrantLock 性能上已经不再是优势，在并发冲突大的场景下，性能上较 Synchronized 差，因此仅仅是 功能上的优势！*

##### 加锁 失败返回

###### 概述

+ **tryLock 方法 加锁 失败返回**

  本质上执行的是 ReentrantLock 自定义的非公平锁 的加锁方式：**nonfairTryAcquire**

  ```
  nonfairTryAcquire 逻辑如下：
   1. 获取 AQS 锁状态
   2. 如果 锁状态 == 0，CAS 获取锁，并将当前线程设置为独占锁的拥有者。返回 true
  	否则下一步！
   3. 如果 锁状态 > 0，判断持有锁的线程 是否为当前线程，如果是，则锁重入，锁状态+1。返回 true
   4. 否则，返回 false
  ```

###### 源码分析

+ **tryLock 方法 加锁 失败返回**

```java
// ReentrantLock

public boolean tryLock() {
    // 获取不公平锁（默认）！
	return sync.nonfairTryAcquire(1);
}

// 获取不公平锁
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    // 获取锁状态（AQS 中的 state）
    int c = getState();
    // c == 0 表示 还没有线程加锁！（c 可以用来实现可重入，c自增即可）
    if (c == 0) {
        // CAS 操作 获取锁！
        if (compareAndSetState(0, acquires)) {
            // 将当前线程 更新为 独占锁的持有者！
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // c != 0 表示 已经有线程加锁，此处判断是否为本线程加的锁！
    else if (current == getExclusiveOwnerThread()) {
        // 锁重入
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // 否则，是其他线程加的锁，此线程加锁失败！
    return false;
}
```

+ *AbstractQueuedSynchronizer*

```java
// 锁状态
private volatile int state;
// 当前独占锁的拥有者
private transient Thread exclusiveOwnerThread;

// CAS 操作 设定 锁状态
protected final boolean compareAndSetState(int expect, int update) {
    // STATE 是 锁状态 属性的 句柄！
    return STATE.compareAndSet(this, expect, update);
}

// 设置独占锁
protected final void setExclusiveOwnerThread(Thread thread) {
    exclusiveOwnerThread = thread;
}
```

##### 加锁 失败阻塞

###### 概述

+ **lock 方法，加锁失败则 被阻塞！**

  1. 首先：并不会直接调用 加锁阻塞 的接口，而是直接尝试加锁（调用自身的 **tryAcquire** 尝试获取锁），如果失败，则再调用 加锁阻塞 的方法（**acquireQueued** 方法）！

     ```
     nonfairTryAcquire 逻辑如下：
     1. 获取 AQS 锁状态
     2. 如果 锁状态 == 0，CAS 获取锁。返回 true
     	否则下一步！
     3. 如果 锁状态 > 0，判断持有锁的线程 是否为当前线程，如果是，则锁重入，锁状态+1。返回 true
     4. 否则，返回 false
     ```

  2. 真正执行 阻塞同步 的方法 acquireQueued

     ```
     acquireQueued 逻辑如下：
     1. 为当前线程绑定一个结点，并将 线程结点 加入 等待 AQS等待队列 的末尾！
     2. 进入自旋

     3. 获取当前线程结点的 前驱结点，判断是否为 AQS等待队列 的首结点，如果是，则尝试获取锁！
     	如果不是，或者获取锁失败，则进行下一步！
     	
     	注意：首结点只是一个标记，与之相关的线程正是当前持有锁的线程）
     	
     4. 判断当前线程是否应该被 阻塞，如果应该被阻塞，阻塞线程！
     	注意：如果是第一次判断的话，那么 当前线程至少还有一次的 自旋机会！
     ```

###### 源码分析

+ *【ReentrantLock】* **lock() 加锁，阻塞同步** 

  可以响应中断！

```java
// ReentrantLock

public void lock() {
    // AbstractQueuedSynchronizer 中的 acquire 方法
   sync.acquire(1);
}
```

+ *AbstractQueuedSynchronizer*

```java
// AbstractQueuedSynchronizer
private transient volatile Node tail; // 等待队列的 尾结点

public final void acquire(int arg) {
    // tryAcquire 方法为 ReentrnatLock 自己实现的 抽象方法
    if (!tryAcquire(arg) &&
        // acquireQueued 方法 AbstractQueuedSynchronizer 中已实现的方法：加锁失败则 阻塞！
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)){
        selfInterrupt();
    }
}

// 加锁：失败则阻塞
final boolean acquireQueued(final Node node, int arg) {
    try {
        boolean interrupted = false;
        // 自旋
        for (;;) {
            // 获取 当前结点的 前驱结点
            final Node p = node.predecessor();
            // 如果 前驱结点 == 等待队列的首结点，则通过 tryAcquire 尝试获取锁
            if (p == head && tryAcquire(arg)) {
                // 成功获取锁，则将该结点置为队列的首结点（作为一个标记）
                setHead(node);
                p.next = null; // help GC
                return true;
             }
            // 否则，说明前面还有线程结点 正在阻塞等待
            
            /*
             · 到这儿，说明前面获取锁失败了，则首先判断是否应该将线程阻塞，防止无谓的自旋！
              	如果 shouldParkAfterFailedAcquire 为 true 
              	则 执行 parkAndCheckInterrupt 阻塞线程，并且可以响应中断！
            */
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } catch (Throwable t) {
        cancelAcquire(node);
        throw t;
    }
}

// 将获取锁的线程结点 置为 等待队列的首结点，作为一个标记
private void setHead(Node node) {
    head = node;
    // 结点绑定线程 置空
    node.thread = null;
    // 结点前继引用 置空
    node.prev = null;
}

// 由 前继结点的waitStatus 决定是否应该 将本线程阻塞！
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    // 判断前驱的 waitStatus，如果为 SIGNAL 表示，当前线程应该被阻塞！（SIGNAL=-1）
    if (ws == Node.SIGNAL)
        // ws = -1
        return true;
    if (ws > 0) {
        // ws >0，对应着 前驱结点 已经放弃获取 锁！
        do {
           node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 否则：其他情况，将前驱 waitStatus 设定为 -1
        // 这对于前继线程结点 而言，后继线程 需要 断开连接（需要被阻塞）
        pred.compareAndSetWaitStatus(ws, Node.SIGNAL);
    }
    // 如果 前继的 waitStatus 没有表明当前线程 是否应该被阻塞，则 当前线程还有自旋的机会！
    // 即：至少再给 一次机会给当前线程 自旋，避免被 阻塞！
    return false;
}

// 阻塞线程：可响应中断
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

+ *Node 类*

```java
// Node 子类

volatile Thread thread;
Node nextWaiter;
static final Node EXCLUSIVE = null;	// 标记当前结点 代表的线程获取 独占锁

// 将线程绑定到 结点中，并将结点加入 等待队列的 末尾！
private Node addWaiter(Node mode) {
    // 为当前线程创建 结点！（将线程绑定到结点）
    Node node = new Node(mode);
    // CAS + 自旋：将 node 设置到 等待队列的末尾
    for (;;) {
        // tail ：AbstractQueuedsynchronizer 中 等待队列的 尾结点！
        Node oldTail = tail;
        if (oldTail != null) {
            // 将当前 node 的前驱设置为 队列的尾结点
            node.setPrevRelaxed(oldTail);
            if (compareAndSetTail(oldTail, node)) {
                oldTail.next = node;
                return node;
            }
        } else {
            initializeSyncQueue();
        }
    }
}
// 构造函数
Node(Node nextWaiter) {
    // 设定下一个等待的 线程结点，默认传入的是 EXCLUSIVE 表示这是一个独占锁！
    this.nextWaiter = nextWaiter;
    /*
     · 将当前线程 与 Node 结点中的 thread 属性绑定
     	THREAD ：代表 Node 中的 thread 属性的 VarHandle 变量，通过 THREAD 就能够操作内存！
     	this ：node
    */
    THREAD.set(this, Thread.currentThread());
}
```

+ *LockSupport 工具类*

```java
// LockSupport 工具类 加锁！
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    U.park(false, 0L);
    setBlocker(t, null);
}
```

##### 加锁 超时等待

###### 概述

+ **tryLock 方法 加锁 超时等待**

  1. 首先：并不会直接调用 超时等待加锁的接口，而是直接尝试加锁（调用自身的 **tryAcquire** 尝试获取锁），如果失败了，则再进入超时等待加锁的 方法（ **doAcquireNanos** 方法）！

     ```
     nonfairTryAcquire 逻辑如下：
     1. 获取 AQS 锁状态
     2. 如果 锁状态 == 0，CAS 获取锁。返回 true
     	否则下一步！
     3. 如果 锁状态 > 0，判断持有锁的线程 是否为当前线程，如果是，则锁重入，锁状态+1。返回 true
     4. 否则，返回 false
     ```

  2. 真正执行 加锁超时等待的 方法 （ **doAcquireNanos** 方法）！

     ```
     doAcquireNanos 主要逻辑：
     1. 为当前线程绑定一个结点，并将 线程结点 加入 等待 AQS等待队列 的末尾！
     2. 进入自旋

     3. 获取当前线程结点的 前驱结点，判断是否为 AQS等待队列 的首结点，如果是，则尝试获取锁！
     	如果不是，或者获取锁失败，则进行下一步！
     	
     	注意：首结点只是一个标记，与指相关的线程正是当前持有锁的线程）
     	
     4. 判断当前线程是否应该被 阻塞！
     	如果确定该线程应该被阻塞，则在阻塞当前线程之前，会判断是否即将达到 所设定的 超时时长！
     	
     	注意：如果是第一次判断的话，那么 当前线程至少还有一次的 自旋机会！
     	
     5. 如果即将超时了，那就不会阻塞当前线程，而是让它自旋重试，直到超时！
     	否则，将当前线程挂起！
     ```

###### 源码分析

+ *【ReentrantLock】* **tryLock(long timeout, TimeUnit unit)** 

  超时等待，可以响应中断！

```java
// ReentrantLock 类

public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

+ *【AbstractQueuedSynchronizer】* **tryAcquireNanos**

```java
public final boolean tryAcquireNanos(int arg, long nanosTimeout) throws InterruptedException {
    // 检查是否已经被中断
    if (Thread.interrupted()){
         throw new InterruptedException();
     }
    // tryAcquire 先尝试加锁，不成功则执行 doAcquireNanos 获取锁
    return tryAcquire(arg) || doAcquireNanos(arg, nanosTimeout);
}
    
// ------------------------- 超时等待获取锁的 核心方法 -------------------------------------
// 超时等待 获取锁！
private boolean doAcquireNanos(int arg, long nanosTimeout) throws InterruptedException {
    if (nanosTimeout <= 0L){
        return false;
     }
    // 获取过期时间
    final long deadline = System.nanoTime() + nanosTimeout;
    // 将当前线程 加入 等待队列的末尾（AQS 中的等待队列），返回当前线程的结点 node
    // 传入 Node.EXCLUSUVE 表示当前线程 需要获取独占锁！
    final Node node = addWaiter(Node.EXCLUSIVE);
    try {
        // 自旋！
        for (;;) {
            // 获取 当前结点的 前驱结点
            final Node p = node.predecessor();
            // 如果 前驱结点 == 等待队列的首结点，则通过 tryAcquire 尝试获取锁
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                return true;
             }
            // 否则，说明前面还有线程结点 正在阻塞等待
            
            // 计算还剩余的 超时时间！
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L) {
                // 已经超时：取消获取锁
                cancelAcquire(node);
                return false;
             }
            /*
             · 如果该线程确实应该被 阻塞，则阻塞之前，再次判断 剩余超时时长，是否大于 1微秒
              		如果大于，则确定阻塞，防止无谓的自旋！
              		否则，说明超时时长马上就到了，此时阻塞线程就得不偿失，自旋一下就可以了！
            */
            if (shouldParkAfterFailedAcquire(p, node) && nanosTimeout > SPIN_FOR_TIMEOUT_THRESHOLD){
                // LockSupport工具类 阻塞线程，并设置超时时长！
                LockSupport.parkNanos(this, nanosTimeout);
             }
            
            // 判断是否被中断
            if (Thread.interrupted()){
                throw new InterruptedException();
             }
        }
    } catch (Throwable t) {
        cancelAcquire(node);
        throw t;
    }
}
// -----------------------------------------------------------------------------------------
【LockSupport】parkNanos 方法是 LockSupport 的静态方法，如下 LockSupport！
```

+ *【LockSupport】* **parkNanos**

```java
// LockSupport 工具类
public static void parkNanos(Object blocker, long nanos) {
    if (nanos > 0) {
        Thread t = Thread.currentThread();
        // 为线程设定 阻塞对象，方便排查跟踪问题，可以从 jstack 线程快照中查看到正在阻塞的对象！
        setBlocker(t, blocker);
        // 调用 Unsafe 工具类阻塞 线程，并且设定超时时间
        U.park(false, nanos);
        setBlocker(t, null);
    }
}
```

+ [LockSupport 底层详解](https://www.jianshu.com/p/ceb8870ef2c5)
+ [UnSafe 解析](https://www.cnblogs.com/wangzhongqiu/p/8441458.html)