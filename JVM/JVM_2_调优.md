### JVM_调优

#### 工具

##### jps

+ **jps** ：输出 主机中的 正在运行的 `虚拟机进程` 的相关信息！

  ```shell
  jps [options] [hostid]
  ```

+ `options`

  ```shell
  -l # 打印：进程的 本地虚拟机唯一ID（LVMID）、进程的 主类全名（若执行的是 JAR 包，则是 JAR 路径）
  -q # 只打印 LVMID，忽略主类名称
  -m # 打印进程启动时传递给主类 main() 方法的参数
  -v # 打印 虚拟机进程启动时的 JVM 参数
  ```

##### jstat

+ **jstat** ：用于 监视虚拟机 各种运行状态信息 的命令行工具！

  ```shell
  jstat [option] vmid [interval [s|ms] ] [count]
  # option 必选
  # vimd # 就是 jps 打印的 LVMID
  # interval count 可选：表示每隔 interval 秒/毫秒，查询 count 次
  ```

+ `option`

  ```shell
  -class # 监视类加载、卸载数量、总空间 以及 类加载所耗费的时间
  -gc # 监视 Java 堆状况、GC 状况
  	# 堆 状况：新生代、老年代、元空间 各个区域的容量 以及 使用量
  	# GC 状况：YGC、FGC 的次数、执行时长等
  	
  -gcutil # 监视内容与 -gc 一样，显示的是 百分比
  -gccause # 与 -gcutil 一样，只是多了 GC 原因

  # ... ...
  ```

  [jstat 工具详解](https://blog.csdn.net/u014756827/article/details/89152795)

##### jinfo

+ **jinfo** ：实时查看 和 调整*虚拟机各项参数*（包括 系统默认参数）

  ```shell
  jinfo <option> <pid>
  ```

+ option

  ```shell
  -flag <name>			# 查看 指定的虚拟机参数
  -flags					# 查看 所有的 虚拟机参数
  -sysprops 				# 查看 系统的默认属性
  <no option>				# 查看 虚拟机参数 和 系统默认参数

  -flag [+|-]<name>		# 启用/屏蔽 name 指定的 虚拟机参数
  -flag <name>=<value> 	# 动态设定 name 指定的虚拟机参数的 值
  ```

##### jmap

+ **jmap**：生成虚拟机的内存快照、查询 finalize 执行队列、堆和方法区的详情 等，可用来分析 jvm物理占用的情况！

  ```shell
  jmap [option] vmid
  ```

+ `option`

  ```shell
  -dump  # 生成转储快照，格式如下
  -dump:[live,]format=b,file<filename> # live 表明是否只 转储（dump） 出存活对象

  -finalizerinfo # 显示 F-Queue 中等待 Finalizer 线程执行的 finalizer 方法的对象 -- Linux/Solaris
  -heap # 显示 堆 详情，例如：回收器类型、参数配置、分代状况 等				  -- Linux/Solaris
  -histo # 显示 堆 中对象的统计信息，包括：类、实例数量、合计容量				 -- Linux/Solaris
  ```

+ 除了 jmap 命令，还可以使用一些 JVM 参数让虚拟机在内存溢出时，自动dump出快照文件！

  ```shell
  -XX:+HeapDumpOnOutMemoryError # 内存溢出时，自动导出内存快照
  -XX:HeapDumpPath=路径 # 指定内存快照时保存的路径
  ```

##### jstack

+ **jstack**  ：Java 堆栈跟踪工具，用于生成 虚拟机当前时刻的**线程快照**（threaddump / javacore 文件）

  线程快照 就是 当前虚拟机 内 每一条线程正在执行的方法堆栈的集合，生成 线程快照的目的 通常是**定位线程 出现长时间 停顿的原因**。`例如：线程死锁、死循环、请求外部资源导致的长时间挂起等`

  *通过 jstack 查看各个线程的调用堆栈，就可以获知 没有响应的线程到底在 后台做什么，或者是等待什么资源！*

  ```shell
  jstack [option] vmid
  ```

+ `option`

  ```	shell
  -F # 当正常输出的请求不被响应时，强制输出线程堆栈
  -l # 除堆栈外，显示关于锁的附加信息
  -m # 如果调用到本地方法栈，可以显示 C/C++ 的堆栈
  ```

#### 思路

+ 线上故障主要与  **CPU、磁盘、内存** 以及 **网络** 等问题相关！

+ 排查时，尽量四个方面一次排查一遍！

  ```
  1. df、free、top 三连
  2. jstack、jmap 具体问题具体分析
  ```

#### 调优

+ [JAVA 线上故障排查：详解 ](https://mp.weixin.qq.com/s/EM5tbVkG4KJZ2D95qjJNuA)
+ [visualvm 使用教程](https://visualvm.github.io/documentation.html)

##### CPU 占用

+ 现象：

  ```
  1. 业务逻辑问题，例如：死循环
  2. 频繁的 GC
  3. 上下文切换过多
  ```

+ 可以使用 jstack 分析堆栈情况

+ **过程：**

  1. *ps 配合 通道+grep* 命令，找到 目标进程！（**也可以使用 jps工具 查看 java 进程**）

     > ps -ef | grep 目标

  2. *top 命令*

     > top -H -p 进程PID  	# 查看 进程中，线程占用 CPU 的情况

     + [top 命令详解](https://www.jianshu.com/p/8a6754f919c5)

  3. *根据 CPU占用率最高的 线程id，到 jstack 中查询对应的堆栈信息*

     + 首先将 线程 的 PID，转换为 16 进制，只是因为 **堆栈中的线程 使用 16进制表示**！

       > printf '%x\n' PID 
       >
       > 输出结果为 nid

     + 到 jstack 中查询对应 nid 的堆栈信息

       > jstack 进程ID | grep '0x123' -C5

     + 找到对应信息后，通常我们会关注 **WAITING 、TIMED_WAITING、BLOCKED** 的三种状态的线程堆栈信息，并对其进行具体分析！

  4. `其次，可以使用如下命令，对 jstack 中的 线程状态 有一个整体把控，如果 WAITING、TIMED_WAITING、BLOCKED 等状态特别多，那么多半是有问题的！`

     > jstack 进程ID| grep "java.lang.Thread.State" | sort -nr | uniq -c

     ```shell
     # 补充 uniq 命令
     uniq 用于 合并 文本文件中重复出现的行列，一般和 sort 命令结合使用
     -c # 参数 用来显示 某行重复出现的 次数
     ```

     + [uniq 命令详解](https://www.runoob.com/linux/linux-comm-uniq.html)

+ **其他**

  + **频繁 gc**

    使用 *jstat* 工具，查看 堆 的 gc 情况

    > jstat -gc 进程ID time\_间隔 		# 每隔 time\_间隔 就打印一次 堆gc 详情！

  + **上下文切换_vmstat**

    使用 *vmstat* 命令，查看 上下文切换，如下示例，*cs (context switch) 属性就是上下文切换次数！*

    > vmstat 进程PID

    ![](image\vmstat.jpg)

    ```shell
    # 补充 vmstat：可以对操作系统的 虚拟内存、进程、CPU 活动进行监控，是对 系统的 整体情况进行统计，不足之处在于，无法堆某个进程进行深入分析
    ```

    + [vmstat 命令详解](https://www.cnblogs.com/ftl1012/p/vmstat.html)

  + **上下文切换_pidstat**

    使用 *pidstat* 命令，针对某个进程进行监控，如下示例，*cswch 和 nvcswch 分别表示 资源切换 和 非自愿切换*

    > pidstat -w 进程ID

    ![](image\pidstat.jpg)

##### 磁盘

+ **df -hl** 命令查看 **磁盘文件系统**状态

  ![](image\df -hl.jpg)

+ **iostat** 命令查看磁盘读写状况：*能定位到具体哪块磁盘出现了问题*

  > iostat -d -k -x

  ![](image\iostat.jpg)

  其中 %util 参数项 表示磁盘的写入程度，rrqpm/s 以及 wrqm/s 分别表示读写速度

##### 内存

+ **OOM**

  1. *Unable to create new native thread*

     ```
      · 无法创建本地线程：没有足够的内存空间 为 线程的Java栈 分配空间！ 
      · 一般而言，可能是线程池 有问题，例如：忘记 shutdown
      	因此，应该首先从 代码 层面寻找问题，使用 jstack 或者 jmap 进行分析！
      
      · 如果一切正常，则可以在 JVM 层面调整 Xss 参数的值，减少单个 thread stack 的大小，容量更多的 线程。
      	另外，可以在 操作系统层面，通过修改 /etc/security/limits.confnofile 和 nproc 来增大 os 对线程的限制！
     ```

  2. *Java heap space*

     ```
      · 由 Java 堆引起的 OOM
      	1. 查找是否有内存泄漏问题，使用 jstack 和 jmap 定位问题！
      	2. 如果一切都正常，则调整 Xmx 参数值，扩大内存
     ```

  3. *Meta space*

     ```
      · 由 元空间 引起的 OOM
      	表示元数据区内存占用 已经达到 XX:MaxMetaspaceSize 设置的最大值，排查思路和上面一样！
     ```

+ **StackOverFlow**

  表示线程的栈 所需 内存 大于 **Xss** 参数值！

  也是使用同样的方法进行排查，没问题的话，再调整 Xss 参数值！

+ *gc 原因*

  内存泄漏问题非常隐蔽，需要关注细节，例如：每次请求都会 new 一个对象，导致大量重复的创建对象！

  进行 文件流 操作但未正确关闭

  手动不当触发 gc

  ByteBuffer 缓存分配不合理 等！

##### GC 问题

+ 堆内 内存泄漏 总是 和 GC 异常相关（当让 GC 还可能引起 CPU负载、网络 等并发症状）

+ 排查

  + 使用 jstat 获取当前分代信息

  + **通过 GC 日志排查问题（常用）**

    > 在 启动参数中 添加如下参数，开启 GC 日志
    >
    >  **-verbose:gc -XX:PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps**

  + 通过 GC 日志，推断出 youngGC 和 fullGC 是否过于频繁或者耗时过长，对症下药！

+ **youngGC 频繁**

  ```
   · yongGC 频繁一般是短周期、小对象较多，先考虑是不是 Eden区/新生代 设置的太小了，看是否能通过 -Xmn -XX:SurvivorRatio 等参数设置解决问题

   · 如果 参数设定之后 youngGC 频率依旧是太高，就需要使用 Jmap 和 dump 文件进一步进行排查了 
  ```

+ **youngGC 耗时过长**

  ```
   · 耗时过长 就需要查看 GC日志 哪一个部分耗时过长
  ```

+ **FullGC** 

  ```
   · FullGC 触发条件：
   	1. system.gc()
   	2. 老年代空间不足：存在大对象、大数组等
   	3. 空间担保失败
   	4. CMS 并发标记失败（G1 也会并发标记失败）
   	5. 永久代空间不足（jdk1.8 开始，就没有这个原因了）
   
   · 根据触发条件 以及 dump的快照，具体分析
  ```


**了解过JVM调优没，基本思路是什么**

```
 · 如果CPU使用率较高，GC频繁且GC时间长，可能就需要JVM调优了。
 · 基本思路就是让每一次GC都回收尽可能多的对象，
 
 · 对于CMS来说，要合理设置年轻代和年老代的大小。该如何确定它们的大小呢？这是一个迭代的过程，可以先采用JVM的默认值，然后通过压测分析GC日志。
	如果看年轻代的内存使用率处在高位，导致频繁的Minor GC，而频繁GC的效率又不高，说明对象没那么快能被回收，这时年轻代可以适当调大一点。
	如果看年老代的内存使用率处在高位，导致频繁的Full GC，这样分两种情况：如果每次Full GC后年老代的内存占用率没有下来，可以怀疑是内存泄漏；如果Full GC后年老代的内存占用率下来了，说明不是内存泄漏，要考虑调大年老代。
 
 · 对于G1收集器来说，可以适当调大Java堆，因为G1收集器采用了局部区域收集策略，单次垃圾收集的时间可控，可以管理较大的Java堆。
```

##### 大佬调优案例

+ [JVM 性能调优](https://www.jianshu.com/p/009df307fbcf)
+ [Visualvm 性能案例分析](https://blog.csdn.net/localhost01/article/details/83422902?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~baidu_landing_v2~default-8-83422902.nonecase&utm_term=visualvm&spm=1000.2123.3001.4430)
+ [jdk 9 部署 visualvm](https://blog.csdn.net/ren9436/article/details/108907596)
+ [visualvm 官网使用手册](https://visualvm.github.io/gettingstarted.html)

