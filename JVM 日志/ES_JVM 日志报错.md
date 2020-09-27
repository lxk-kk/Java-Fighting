##### Minor GC

**[GC (Allocation Failure) 2020-03-22T02:31:00.143+0800: 1.997:  [ParNew Desired survivor size 4358144 bytes, new threshold 1 (max 6) - age   1:    8699184 bytes,    8699184 total : 68160K->8512K(76672K), 0.0254723 secs] 68160K->15548K(1040064K), 0.0256159 secs] [Times: user=0.03 sys=0.00, real=0.03 secs]**

+ ParNew Desired survivor size 4358144 bytes ：4 M

+   new threshold 1 (max 6) - age   1:    8699184 bytes,    8699184 total 

  +  threshold 1 (max 6) - age   1 ：年龄阈值 1 （最大为 6）
  + 8699184 bytes ：age 1 后面的这个数 表示 当前年龄为 ==1 的所有对象占用的内存大小！
  + 8699184 total ：表示年龄 <= 1 的对象总共占用的内存大小！

+ 68160K->8512K(76672K),  0.0254723 secs] 

  + 68160K：当前代 GC 前使用 66M
  + 8512K ：当前代 GC 后使用 8.3M
  + 76672K ：当前代 总量：74.8 M
  + 0.0254723 secs ：GC 耗时
  + 本代：GC 减少了 66-8.3=57.7M

+ 68160K->15548K(1040064K), 0.0256159 secs]

  + 68160K：堆 GC 前使用：66M
  + 15548K：堆 GC 后使用：15.2M
  + 1040064K ：堆总量：1 G
  +  0.0256159 secs：GC 耗时
  + 堆 ：GC 减少了 66-15.2=50.8 M

  新生代减少量 - 堆减少量 = 晋升量：57.7 - 50.8 =  7M

##### Full GC

```
[GC (CMS Initial Mark) [1 CMS-initial-mark: 27058K(963392K)] 32938K(1040064K), 0.0025616 secs] 
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.016/0.016 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.006/0.006 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]

[GC (CMS Final Remark) [YG occupancy: 5880 K (76672 K)]2020-03-24T03:10:32.359+0800: 408.217: [Rescan (parallel) , 0.0365+0800: 408.222: [weak refs processing, 0.0001069 secs]2020-03-24T03:10:32.365+0800: 408.222: [class unloading, 0.0061206 secs]2020-03-24T03:10:32.371+08000.0052191 secs]2020-03-24T03:10:32.376+0800: 408.234: [scrub string table, 0.0009227 secs][1 CMS-remark: 27058K(963392K)] 32938K(1040064K), 0.0186662 secs] [0.02 secs]
2020-03-24T03:10:32.378+0800: 408.235: Total time for which application threads were stopped: 0.0188166 seconds, Stopping threads took: 0.0000440 seconds
2020-03-24T03:10:32.390+0800: 408.248: [CMS-concurrent-sweep-start]
2020-03-24T03:10:32.397+0800: 408.254: [CMS-concurrent-sweep: 0.007/0.007 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]
2020-03-24T03:10:32.397+0800: 408.255: [CMS-concurrent-reset-start]
2020-03-24T03:10:32.399+0800: 408.257: [CMS-concurrent-reset: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

```



[JVM 调优参数](https://www.jianshu.com/p/0cafcb8319ab)

```
Java HotSpot(TM) 64-Bit Server VM (25.121-b13) for linux-amd64 JRE (1.8.0_121-b13), built on Dec 12 2016 16:36:53 by "java_re" with gcc 4.3.0 20080428 (Red Hat 4.3.0-8)
Memory: 4k page, physical 1882080k(1165448k free), swap 1049596k(1049596k free)
CommandLine flags: 
-XX:+AlwaysPreTouch 
-XX:CMSInitiatingOccupancyFraction=75
-XX:ErrorFile=logs/hs_err_pid%p.log
-XX:GCLogFileSize=67108864 
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=data 
-XX:InitialHeapSize=1073741824
-XX:MaxHeapSize=1073741824 
-XX:MaxNewSize=87244800 
-XX:MaxTenuringThreshold=6 
-XX:NewSize=87244800 
-XX:NumberOfGCLogFiles=32 
-XX:OldPLABSize=16 
-XX:OldSize=174489600 
-XX:-OmitStackTraceInFastThrow 
-XX:+PrintGC -XX:+PrintGCApplicationStoppedTime 
-XX:+PrintGCDateStamps 
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps 
-XX:+PrintTenuringDistribution 
-XX:ThreadStackSize=1024 
-XX:+UseCMSInitiatingOccupancyOnly 
-XX:+UseCompressedClassPointers
-XX:+UseCompressedOops 
-XX:+UseConcMarkSweepGC 
-XX:+UseGCLogFileRotation 
-XX:+UseParNewGC
```

![](image\ES报错.png)