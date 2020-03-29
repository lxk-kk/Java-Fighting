#### 【Redis_1_基础】

[电子书：Redis 设计与实现](redisbook.com/)

[Redis 中文官网](http://redis.cn/)

##### Redis 概述

###### Redis 是什么？

+ Redis 是一个基于 内存的 高性能 非关系型数据库！

###### Redis 优点？

+ redis 的数据读写都是内存级别，相比于 传统的将数据保存到磁盘的 数据库而言 redis 的 数据读写性能很出色！
+ redis 出色之处不只是性能，它最大的魅力在于，它具有字符串（String）、列表（List）、集合（Set）、有序集合（Sorted Set）、hash 五种数据结构，这些数据结构都是以 key-value 的形式操作数据，并且每一种数据结构都有特定应用场景！

###### Redis 为什么这么快？

1.  Redis 的数据存在 内存中，数据操作`完全基于内存`，所以数据访问非常快速！

2. redis 底层的数据结构有 hash table、跳表、压缩表等，当数据量较小时，会首先尝试采取压缩表的形式存储，达到一定限度后就会转换为 跳表 或者 哈希表。压缩表的时间复杂度为 O(N)，但是数据量小时，仍然可以视为 O(1)，而 哈希表 访问元素就是 O(1)，所以非常快！

   （压缩表：是为了优化内存占用，但是数据访问是 O(N)，所以需要转换。[详见：Redis 压缩表 ziplist 解析](https://blog.csdn.net/qiangzhenyi1207/article/details/80353104)）

3. Redis 是单线程处理任务的，避免了上下文切换以及多线程之间的资源竞争问题。

4. 不再需要用锁保证线程安全性，也就不存在`获取锁、释放锁、线程阻塞、线程唤醒`等问题，就更不会出现`死锁`导致CPU性能消耗！

5. 在 网络IO上，redis 采用**IO多路复用**，它底层实现了`evport、kqueue、select、epoll`等多种技术，并根据所属平台的不同，选出性能的最高的一种！

   *多路：多个网络连接（多个 Socket 连接），复用：指复用同一个线程处理请求。*
   
   linux平台，默认使用 **epoll **技术，将 网络IO 耗时降到最低，使得单个线程能够高效的处理并发的 socket 请求！
   
   [ IO 多路复用技术 epoll：底层原理](https://www.jianshu.com/p/e6b9481ca754)

###### Redis 处理高并发？

1. Redis 将数据存在内存中，所有数据操作都是基于内存的，所以数据访问非常快！

2. Redis 是单线程的，由于没有多线程的下的上下文切换，以及资源竞争开销，也没有资源同步带来的开销，所以，单线程下表现的很好。若多核CPU，则可以采用 单线程+多进程的方案，也就是主从、集群等方案提高CPU利用率！

3. 在网络层，redis 采用 多路复用技术，使得单线程下也能高效的处理多个 socket 请求！

   多路复用技术 是`非阻塞IO`，其内部采用了` epoll+自己实现的简单的事件框架`，将` epoll 中的读、写、关闭、连接`等都转化成了事件，然后利用 `epoll 的多路复用性`，将网络IO 的耗时降到最低！

###### Redis 为什么是单线程？

1. Redis 是基于内存的数据操作，CPU不会是 Redis 的瓶颈（不需要磁盘IO？），redis 的瓶颈可能会是 机器内存的大小或者网络带宽，既然 单线程容易实现，而且 CPU 不会称为瓶颈，那就顺理成章的采用单线程方案！

2. 单线程 减少 CPU 消耗

   采用单线程， 就避免了不必要的`上下文切换`和`资源竞争`。

3. 单线程下不需要各种锁的性能消耗

   Redis 的数据结构并不全是简单的 key-value ，还有 list，hash等复杂的结构，这些结构有可能会进行很细粒度的操作，比如在很长的列表中添加一个元素，或者在 hash 种添加删除 一个对象，若采用多线程实现，则这些操作都需要加锁，并发量大时，同步开销也会很大，例如`加锁、解锁、线程阻塞、线程唤醒等甚至还有死锁`，得不偿失！

4. 单线程多进程集群方案

   在多核CPU中，为了能更好的利用 CPU 资源，可以采用`主从、集群`方案，能保证高可用的同时，还能充分利用 CPU 资源！这就是 单线程多进程方案！

5. 网络层：IO 多路复用技术

   在 网络IO 上，redis 采取**IO多路复用**，它在底层实现了`evport、kqueue、select、epoll`等技术，并根据所属平台的不同，选择性能最高的一个，作为 网络IO。

   *多路：多个网络连接（多个 Socket 连接），复用：指复用同一个线程处理请求。*

   在 linux 平台种，默认选择**epoll**技术，将 网络IO 耗时降到最低，使单个线程能够高地处理并发的 socket 请求！

   [ IO 多路复用技术 epoll：底层原理](https://www.jianshu.com/p/e6b9481ca754)

###### 为什么使用 Redis？

+ 第一：博客可以认为是读多写少的应用，由于 redis 数据访问速度比关系性数据库要快很多，所以，我使用 redis 作为第三方缓存，这样对于热点博客，就可以避免 频繁地 读取数据库，进而加速了应用的响应速率，也减轻了数据库的访问压力！
+ 第二：redis 支持 丰富的数据类型，分别是：字符串、列表、集合、有序集合、hash 等5种数据类型，我就正好使用了 有序集合这个数据结构，实现我博客首页的排行榜功能！而之前说的 缓存，就是使用的 字符串 这个数据结构！

##### Reids vs memcache

```java
/*
 · memcache 是一个高性能的 分布式 内存对象缓存系统！
 
 · 相同点：
 	1、两者都是非关系型内存数据库
 
 · 区别：
 	*  持久性：
 	  memchache 不支持持久化，并且数据所占用空间不能超过设定的内存大小，一旦 memcache 挂机，则数据丢失不可恢复！
 	  redis 有持久化机制：RDB快照、AOF日志；服务器挂机时，redis 可以通过 aof 日志文件恢复数据！
 	  
 	  ？？？虚拟内存：被废止？redis 当物理内存用完后，可以将一些很久没有用到的 value 交换到 磁盘中，而在内存中保留 key 信息，这使得redis可以存储超过其自身机器内存大小的数据！（Redis 内存管理机制！！！）
 
   *  数据类型
     redis 支持 5种 数据类型，而memcache只支持简单数据类型，需要服务端自己处理复杂的对象！
   
   *  分布式存储
     redis 支持 master-slave 主从复制模式，能够横向扩展成集群！
     memcache 使用 一致性hash 做分布式！
*/
```

+ [redis 与 memcache 的不同](https://www.jb51.net/article/62506.htm)

##### Redis 对象处理机制

+ redis 为兼容其 不同类型的指令（String、List、Hash 等类型的指令），自定义了一套 类型系统

  ```c
  /*
  * Redis 对象
  */
  typedef struct redisObject {
      // 类型：key 的类型：String、List、Set、Hash、Sorted Set等
      unsigned type:4;
      // 对齐位
      unsigned notused:2;
      // 编码方式：value 的存储方式
      unsigned encoding:4;
      // LRU 时间（相对于 server.lruclock）
      unsigned lru:22;
      // 引用计数：释放对象
      int refcount;
      // 指向对象的值
      void *ptr;
  } robj;
  
  /*
   · 最重要的三个属性：type、encoding、ptr
  	
   · type：记录 采用的 Redis 类型（如下）
  		对于 不同的类型，有些类型可以使用同一个指令，只是内部实现不同，有些 类型有独有指令！
  		使用 type 记录类型，方便 Redis 利用多态获取合适 的指令函数！
  */
  
  #define REDIS_STRING 0 // 字符串
  #define REDIS_LIST 1 // 列表
  #define REDIS_SET 2 // 集合
  #define REDIS_ZSET 3 // 有序集
  #define REDIS_HASH 4 // 哈希表
  
  
  /*
   · encoding：记录 value 的编码格式 —— 每一种编码格式都对应一种 数据结构！
  		对于 Redis 中的一个类型，会使用多种 格式存储 value，是为了在 时间和空间上的均衡！
  */
  #define REDIS_ENCODING_RAW 0 // 编码为字符串
  #define REDIS_ENCODING_INT 1 // 编码为整数
  #define REDIS_ENCODING_HT 2 // 编码为哈希表
  #define REDIS_ENCODING_ZIPMAP 3 // 编码为 zipmap
  #define REDIS_ENCODING_LINKEDLIST 4 // 编码为双端链表
  #define REDIS_ENCODING_ZIPLIST 5 // 编码为压缩列表
  #define REDIS_ENCODING_INTSET 6 // 编码为整数集合
  #define REDIS_ENCODING_SKIPLIST 7 // 编码为跳跃表
  
  /*
   · ptr 是一个指针，指向实际保存值的数据结构，这个数据结构由 type 和 encoding 共同决定
  */
  
  /*
   · refcount ：引用计数
   	c 语言并不具备自动的内存回收功能，所以在自建的 对象系统中 构建了一个 引用计数回收机制。通过这个机制，程序可以通过 跟踪 对象的引用计数信息，在适当的时候自动释放对象，并进行内存回收！
  
   · refcount 引用计数的状态变化
   	* 在创建一个新对象时，引用计数值初始置 = 1
   	* 对象被一个新程序使用时，引用计数值 + 1
   	* 对象不再被一个程序使用时，引用计数值 - 1
   	* 当对象引用计数的值减为 0 时，对象被所占用的空间被释放！
  */
  ```

  ![](image\对象处理机制.png)

##### Redis 数据类型 vs 应用

+ redis 中的数据结构：String、List、Set、Sorted Set、Hash

###### String

+ String 是最简单的数据类型，可以理解为 和 memcache 一样的类型，key-value 结构！

+ String 是二进制安全的，大小上限为 1GB，也就是说 Redis 中的 String 能存储任意格式的二进制数据：jpg图片、序列化后的对象等

  ```c
  /*
   · 字符串类型分别使用 REDIS_ENCODING_INT 和 REDIS_ENCODING_RAW 两种编码：
   		REDIS_ENCODING_INT 使用 long 类型来保存 long 类型值。
   		REDIS_ENCODING_RAW 则使用 sdshdr 结构来保存 sds （也即是 char* )、long long 、double 和 long double 类型值。
   	
   · 在 Redis 中，只有能表示为 long 类型的值，才会以整数的形式保存，其他类型的整数、小数和字符串，都是用 sdshdr 结构来保存。
  */
  
  /*
   · 为什么 Redis 中的 String 是二进制安全的？（底层原理）
  */
  struct sdshdr {
      // buf 长度
      long len;
      // 剩余可用长度
      long free;
      // c 语言中的 char 是 1个 字节，相当于 byte
      char buf[];
  };
  /*
   · 解析：
   	Redis 中的 字符串处理，比 c 语言高效，并且是二进制安全的！
  
   · 二进制安全：程序不会对数据格式进行修改！
   	所有 sds API 都会以 二进制的形式处理 sds 中的 buf数组，程序不会对数据做任何限制、过滤、或者假设，也就是说数据写入时是什么样的，读取时就会是什么样的。因此 buf数组 不是用来保存字符串的，而是用来保存 二进制数据的！
   	其次，sds 是通过 len 属性判断数组长度的，而并非使用 空字符 判断数组长度！（不过，为了兼容 C 字符串函数，sds 依旧以 空字符结尾，不计入 len）
   	而 c 语言中的 字符串，是以 '\0' 为结束符的，一旦遍历到字符串中包含了结束符，则其之后的字符将被忽略掉！若在 Redis 中使用 c 语言中的字符串就会导致 信息丢失！
   	
   	通过使用 二进制安全的 sds 结构，而不是 c 语言的字符串，使得 redis 不仅可以保存文本，还可以保存任意格式的二进制格式：jpg等！
  
   
   · 为什么处理比 c 语言中的 字符串高效？
   	正是由于 sds 有专门记录字符串长度的，所以拼接字符串时，直接在剩余空间中追加字符串即可，获取字符串长度时也可以直接返回，而不需要像 c 语言一样遍历字符串！
  */
  
  /* 
   · sds 与 c语言字符串的区别：
   	* 常数时间获取字符串长度
   	* 杜绝缓冲去溢出：有 len、free 记录；而 c 字符串即不记录长度，也不会知道剩余空间，就更不会知道是否分配了足够的空间操作数据！拼接字符串时可能出现空间溢出！
   	* 减少了修改字符串时的内存重分配：
   		1、空间预留分配：扩展字符时，不仅分配需要的空间，还会多分配空间备用
   			（以 1M 为界限，扩展后len 1M以内，分配等同 len 的 free 空间；1M 以外 分配 1M free 空间）！
   		2、惰性空间分配：删除时，并不着急缩短，而是用 free 记录剩余空间，以备后用
   	* 二进制安全
   	* sds 兼容 部分 c字符换 函数：buf 依旧是以 空字符结尾，不计入 len（避免重写代码）
  */
  ```

+ 应用

  1. **缓存**

     String 字符串是最基本的数据类型，利用 redis 作为缓存，配合其他数据库作为 持久层，借助 redis 支持高并发的特点，可以大大加快应用的响应速度，同时降低后端数据库的压力！

  2. **计数器**

     String 类型 提供了 自增自减的指令。

     许多系统都会使用 Redis 作为系统的实时计数器，可以快速实现计数和查询功能，而且最终的数据结果可以按照特定的时间落地到数据库或者其他存储介质当中进行永久存储！

  3. **共享用户 session**

     多服务系统下，用户在各个服务之间穿梭，就会导致 session 覆盖的问题，因此，可以借助 redis 作为第三方的 session 共享机制，由于 redis 性能高，所以身份信息获取会很快速！

###### List

+ List 是一个 `双端链表` 结构，主要功能就是 push 、pop、获取一个范围内的所有元素等等！

+ 常用命令：*lpush / rpush / lpop / rpop / lrange / lrem* 。`lrange ：读取某个范围内的元素; lrem ：从 左/右 删除指定个数的 匹配到的 value，或者全部删除匹配到的 value。`

+ List 使用 底层有两种实现方式：`REDIS_ENCODING_ZIPLIST（压缩表）、REDIS_ENCODING_LINKEDLIST（链表）`

  ```c
  /*
   · 创建新列表时，Redis 默认使用 REDIS_ENCODING_ZIPLIST（压缩表）的结构储存数据!
   
   · 结构转换
   	当以下 任意一个条件满足时，会采用 REDIS_ENCODING_LINKEDLIST（双端链表） 结构存储数据！
  		1、试图往 列表中添加一个 字符串，并且字符串的从长度超过 server,list_max_ziplist_value（ 默认值为 64MB ）
  		2、ziplist 包含的结点超过 server.list_max_ziplist_entires（ 默认值为 512个 ）
  */
  ```

+ 阻塞原语：*blpop / brpop / brpoplpush*

  *blpop list1 list2 timeout：可设置超时时间，参数可以是多个 List，但只会作用一个 List。从左到右，若 List 不为空，则执行 lpop，若全部为空，则 阻塞 timeout 时长，超时发生 返回 nil，timeout=0 表示 一直阻塞。（ brpop 同 blpop，不同在于 会执行 rpop ！）*

  ```c
  /*
   · 阻塞的条件：blpop、brpop、brpoplpush 三个命令都可能造成 客户端阻塞
   · 阻塞原语不一定会造成客户端阻塞：
   	1、只有当这些命令用于空列表时，他们才会阻塞客户端！
   	2、如果被处理的列表不为空，他们就执行阻塞版本的 lpop、rpop、rpoplpush 命令！
   
   · 阻塞客户但步骤：
   	1、将客户端状态设置为“正在阻塞”，并记录阻塞这个客户端的各个键，以及阻塞的最长时限 timeout 等数据！
   	2、将客户端信息记录到 server.db[i]->blocking_keys 中（db[i] 客户端所访问的数据库，redis 中默认 16 个库）
   	3、继续维持客户端和服务端之间的 网络连接，但不再像客户端传送任何信息，造成客户端阻塞！
   	
   	上述 blocking_keys 是一个字典，字典的 key 是造成客户端阻塞的 list_name，value 是一个链表，存放所有被 key 阻塞的 客户端信息！
  
   · 客户端 脱离阻塞状态 的三种方法：
   	1、被动脱离：阻塞因 push、insert 等添加命令而被取消。
   	  Redis 采用 FBFS（先阻塞先服务）的策略，当 List 有新元素添加，则可以让 被这个 list 阻塞的客户端 脱离阻塞的状态，脱离的客户端数量，取决于添加新元素的数量，每添加一个 元素，只会 取消阻塞一个 客户端，并且先被阻塞客户端先获取元素执行！
     
     2、主动脱离：阻塞因 达到设定的阻塞时长 被取消！（每个客户端都会为阻塞原语设定 阻塞时限）
     3、强制脱离：客户端强制终止和服务器的连接，或者服务器停机！
  */
  ```

+ 应用

  + **列表**（最基础）

    关注列表、粉丝列表、评论列表、点赞列表等

  + **Redis做异步队列**

    List 是一个双端队列，可以实现任意方向的`阻塞队列：左进右出的命令组合、右进左出的命令组合`等

    使用 List 作为队列，指令 rpush 生产消息，lpop 消费消息，当 lpop 没有消息时，要适当 *sleep* 一会儿再重试！List 还有指令 *blpop / brpop* ，有消息时直接消费，没有消息的时候，阻塞直到消息到来！*使用 `blpop / brpop` 避免了  `线程 sleep 并轮询` 的资源开销！*

    *结合 Pub/Sub 主题订阅模式，可以实现  1:N 的消息队列*，不过当 消费者下线的情况下，生产的消息会丢失，建议使用 专业的 MQ（RabbitMQ 了解一下）

  + **文章列表 或者 数据分页展示**

    可以基于 List 实现分页查询，这个功能非常棒，基于 Redis 做简单的高性能分页，可以应用在 微博 那种下拉不断分页的场景，性能高！

###### Hash

+ Redis hash 是一个 string 类型的 field 和 value 的映射表，元素的添加、删除、访问等操作都是 O(1) 时间！

+ hash 特别适合用于存储对象。相较于 将对象中的每个字段 单个作为上述 String 类型存储时，将一个对象 存在 Hash 中会占用更少的内存，并且更方便的存取整个对象。

+ Redis hash 有两种实现方式：`REDIS_ENCODING_ZIPLIST（压缩表）、REDIS_ENCODING_HT（哈希表）`

  ```c
  /*
   · 创建空白哈希表时，程序默认使用 REDIS_ENCODING_ZIPLIST（压缩表） 编码！
   
   · 结构转换
   	当以下任何一个条件被满足时，程序将编码将切换为 REDIS_ENCODING_HT（哈希表）
  	  1、哈希表中某个键或某个值的长度大于 server.hash_max_ziplist_value （默认值：64M）
  	  2、压缩列表中的节点数量大于 server.hash_max_ziplist_entries （默认值：512个）。
    
   · 为什么将一个对象存放在 Hash 中会占用更少的内存？
   	如上述，Hash 会首先以 压缩表的编码格式存放数据，当 entry 大于 512个时，或者 value 值占用空间超过 64MB 时，会自动转换为 Hash 表！对于一般对象而言，其大小和属性个数都不会超过这个默认值！所以，将对象存放在 Hash 中，大多会使用 压缩表的形式存储！
   	压缩列表是 Redis 为了节约内存而开发的， 由一系列特殊编码的连续内存块组成的顺序型（sequential）数据结构。一个压缩列表可以包含任意多个节点（entry）， 每个节点可以保存一个字节数组或者一个整数值。
   	
   · Hash 表初始大小为 4 个 dictEntry
  */
  ```

  + [详见：Redis 压缩表 ziplist 解析](https://blog.csdn.net/qiangzhenyi1207/article/details/80353104)

+ Hash 算法

  ```c
  // 【补充】 redis 字典结构（hash）
  typedef struct dict {
      // 类型特定函数
      dictType *type;
  
      // 私有数据
      void *privdata;
  
      // 哈希表
      // ht ：包含了两个 哈希表；
      // 一般情况下，字典只使用 ht[0] 作为哈希表；ht[1] 哈希表 只会在对 ht[0] 进行 rehash 时使用！
      dictht ht[2];
  
      // rehash 索引：是 hash 表进行 reahsh 时的进度
      // 当 rehash 不在进行时，值为 -1
      int rehashidx; 
  } dict;
  
  // 【补充】 字典中的 哈希表结构
  typedef struct dictht {
      // hash结点 二维数组：因为hash 冲突时，会产生链表
      dictEntry **table;
      // 哈希表大小
      unsigned long size;
      
      // 哈希表大小掩码，用于计算索引值
      // 总是等于 size - 1
      unsigned long sizemask;
  
      // 该哈希表已有节点的数量
      unsigned long used;
  } dictht;
  
  // ---------------------------------------------------------------
  // hash 算法
  
  // 使用字典 设置的 hash 函数，计算 key 的哈希值
  // hash 函数 使用了 MurmurHash2 算法
  hash = dict->type->hashFunction(key);
  
  // 使用 hash 表的 sizemark 属性 与 上述 hash 值，取余，计算出 索引
  // ht[x] 中的 x 根据情况可能是 1 也可能是 0
  index = hash & dict->ht[x].sizemark;
  ```

  + Redis 使用 MurmurHash2 算法来计算 hash key 的 hash 值。这种算法的优点在于，即使输入的键是有规律的，算法仍然能给出一个很好的随机分布性，并且算法速度很快！

+ 解决键冲突

  当时上述 hash 算法将多个键分配到同一个 索引上时，便称这些键发生了冲突！

  ```c
  // 【补充】 hash 结点结构
  typedef struct dictEntry {
      // 键
      void *key;
      // 值
      union {
          void *val;
          uint64_t u64;
          int64_t s64;
      } v;
      // 指向下个哈希表节点，形成链表：解决键冲突
      struct dictEntry *next;
  } dictEntry;
  ```

  Redis 采用*链地址法* 来解决冲突，每个 hash 结点都有一个 next 指针，多个 hash表结点就可以用 next 指针构成一个 单向链表，本分配到同一个索引上的多个结点就可以用这个单向链表存储，这就解决了 键 冲突问题（像 jdk1.7 的 hashmap 一样）

  与 hashmap jdk1.7 一样，结点采用`头插法 插入`：jdk1.7 是多线程的，并且线程不安全，所以 jdk1.8 会采用 `尾插法`，而对于 单线程的 redis 来说，并不会出现 结点之间循环引用的问题，并且*头插法有两个好处* ：`1、不需要额外维护链表尾部结点，并且插入操作方便；2、对于一个数据库来说，最近插入的数据可能会被频繁的获取，所以头插法能节省查找耗时`

  ![](image\hash 键冲突.png)

+ rehash & 扩容 & 缩容

  **扩容缩容的目的：保证负载因子可以维持在一个合理的范围内 —— 我认为可以减少散列冲突，提高数据访问效率！**随着插入操作的不断增加，hash 表中存入的 键值对越来越多，当 哈希表被占满时，这之后插入的 结点都会通过拉链法的方式解决冲突，相应的 访问（修改、查询、删除）结点效率变低，因为不仅需要定位到 hash 表中的某个结点，还需要对该结点的单向链表进行遍历，链表操作需要 O(N)！

  Redis 通过动态变化的 `负载因子` 判断是否有必要扩容和缩容，解决上述问题！

  ```c
  // 负载因子的计算
  // 负载因子 = 哈希表已保存节点数量 / 哈希表大小
  load_factor = ht[0].used / ht[0].size
  ```

  *何时扩容/缩容？*

  ```c
  /*
   · 扩容条件：
   	* 服务器目前没有执行 BGSAVE 指令 或 BGREWRITEAOF 指令，并且 哈希表负载因子 >= 1（即：use=size 刚好被占满，未出现冲突）
   	* 服务器目前正在执行 BGSAVE 指令 或 BGREWRITEAOF 指令，并且 哈希表负载因子 >= 5 (即：use=5*size 平均情况：每个表结点上都有长度为 5 的单向链表)
  
   · 缩容条件：
   	* 当 哈希表 负载因子 < 0.1 时，程序自动对 哈希表执行收容操作！
  
   · 满足条件后，自动扩容/缩容！
   
   · 为何 扩容时 两种情况中的 负载因子 不一样？【见如下】
   	因为 在执行 BGSABE、BGREWRITEFOA 命令的过程中，Redis 需要创建当前服务器的子进程，而大多数操作系统都采用 写时复制（copy-on-write）技术来优化子进程的使用效率，所以在子进程存在期间，服务器会提高执行 扩容 所需要的负载因子，从而尽可能的避免在子进程存在期件进行 hash 表扩容，这可以避免不必要的内存写入，最大限度的节约内存（写时复制的内存）！
  */
  ```

  *如何 扩容 / 缩容*

  + 扩容、缩容 哈希表的工作是通过执行 rehash（重新散列）操作来完成的！

  *Rehash*

  **rehash 步骤如下：**（如下图）

  1. 为字典 ht[1] 哈希表分配新空间`（新空间的大小由 原 哈希表中的结点总数决定！）`

     * 扩容：ht[1].size = 第一个 `大于等于` ht[0].used * 2 的 `2的幂次`。`（例 used=6，则 6*2=12 < 16 = 2^4，所以 ht[1].size = 16）`
     * 缩容：ht[1].size = 第一个 `大于等于` ht[0].used 的 `2的幂次`。`（例 used = 6 ，则 6 < 8 = 2^3，所以 ht[1].size = 8）`

  2. 将保存在 ht[0] 中的所有键值对 rehash 到 ht[1] 上。**这是个 渐进式 Rehash 的过程**

     rehash 指：重新计算 key 的 hash值，并对新的 size 取模，计算 索引值（index），然后将 key-value 对放到 ht[1] 哈希表对应的索引位上！

  3. 当 ht[0] 包含的所有 键值对 都迁移到 ht[1] 上时，ht[0] 变为空表；此时，释放 ht[0]，并将 ht[1] 设置成 ht[0]，并在 ht[1] 上新创建一个 空白 哈希表，为下一次 rehash 做准备！

  ![](image\rehash 过程.png)

  *渐进式 Rehash*

  + 如上述 需要将 ht[0] 中的 键值对 全部迁移到 ht[1] 表中，但是这个迁移任务并不是 一次性、集中式地完成的！而是 分批次、渐进式的完成！

    假设 ：ht[0] 中有大量的 key-value 对。

    *一次性集中式 rehash*  会造成 庞大的计算量，导致 CPU密集型计算，服务器会在一段时间内 *暂停服务* ！

    而 **渐进式 Rehash 采用 分而治之 的思想，将庞大的计算压力 分摊到 每一次的 数据访问（增删查改）上！**

  + **渐进式 Rehash 步骤：**（如下图）

    1. 为 ht[1] 分配空间时，字典会同时持有 ht[0]、ht[1] 两个哈希表！

    2. 在 字典中维护一个` 索引计数器变量 rehashidx`，并将它的值设置成 0 。

       rehashidx ：表示`下一次 rehash ht[0].table[rehashidx] 链表中的所有键值对！`

       将 rehashidx 设置为 0，表示 rehash 工作正式开始！

    3. 在 rehash 期间，redis 每次对 字典执行 添加、删除、查找、更新 等操作时，程序除了执行指定的操作以外，还会根据 rehashidx 将链表上的所有 键值对 rehash 到 ht[1] 上。当此次 rehash 完成，则 rehashidx + 1，为下一个 rehash 链表 做准备！

       `对字典的操作：`

       ​	`查找：会首先在 ht[0] 哈希表上进行，若没找到则在 ht[1] 上进行！`

       ​	`删除、更新：ht[0] 、ht[1] 同时进行！`

       ​	`添加：直接在 ht[1] 新 哈希表上添加！`

       上述措施，**保证了 ht[0] 包含的 键值对数量 只减不增，并随着 rehash 操作的执行而最终变成空表！**

    4. 随着 字典操作的不断执行，最终某个时间点上，ht[0] 中所有的键值对都会被 rehash 到 ht[1] 上，这时 `rehashidx 将被设置成 -1`，表示 rehash 完成！

  ![](image\渐进式 rehash_1.png)

  ![](image\渐进式 rehash_2.png)

###### Set

+ set  是无序集合，会自动去重。它和数学中的 集合相似，Redis 提供了 添加（*sadd*）、删除（*spop*）、求交集（*sinter*）、并集（*sunion*）、差集（*sdiff*）、查看元素是否存在（*sismember*）等操作！

+ 和 List 一样，最大可包含 2^32 个元素

+ Set 底层有两种实现方式：`REDIS_ENCODING_INTSET（整型集合）、REDIS_ENCODING_HT（哈希表）`

  ```c
  /*
   · 当第一个元素添加到集合的时候，决定了集合元素所使用的结构：
   	1、若 第一个元素为为 long long 类型，也就是整数，则初始时刻 为 REDIS_ENCODING_INTSET（整数集合） 结构
   	2、若 不是整数，则 初始时刻使用 REDIS_ENCODING_HT（哈希表）结构
  
  · 结构转换
  	如果初始时刻使用的 REDIS_ENCODING_INTSET（整数集合），则 满足以下条件时，会转换为 哈希表
  	1、试图往 集合中添加一个不能被 long long 表示的类型（也就是不是整数）
  	2、当 集合中的整数个数超过 server.set_max_intset_entries（默认值为 512个）
  
   · REDIS_ENCODING_HT
  	若使用 哈希表，则使用的数据结构和 Hash 类型中的一样！不同之处在于 key 用于存放元素，value 为 null！
  	
  	使用 哈希表 能够轻易的实现 判重、交集、并集、差集等功能！
  */
  ```

+ 应用

  1. **去重**（最基本）
  
     在单机情况下，可以使用 Java 中的 HashSet 做数据去重，而在分布式系统中，Redis 的 Set 可以做全局的数据去重！
  
  2. **交集**（*sinter*）
  
     QQ查看共同好友，可以求交集！
  
  3. **并集**（*sunion*）
  
     合并通讯录：手机上通讯录分为 sim卡联系人 以及 手机联系人，需要合并两种联系人时就可以求并集！
  
     合并所有 blog 标签！
  
  4. **差集**（*sdiff*）
  
     可以实现 好友推荐的功能（向你推荐，你没有但是我有的好友，就可以取差集）

###### Sorted Set

+ sorted set 是 set 的一个升级版本，它在 set 的基础上添加了一个 *double类型的 顺序属性 score*，该元素可以在 添加、修改 元素时指定，每次指定后，zset 会自动重新按照新的值调整顺序！
+ 常用指令
  + *zadd*：添加元素，并指定 score
  + *zincrby*：增加 member 对应的 score 值（自动排序），并返回新的 score
  + *zrank、zrevrank*：获取*指定元素排名*（zrank：score 升序 ； zrevrank：score 降序）
  + *zrange、zrevrange*：获取*指定  哈希表索引 index 区间内有序集合*（zrange：score 升序 ；zrevrange：score 降序）
  + *zrangebyscore、zrevrangebyscore*：获取*指定 socre 区间内的有序集合*（zrangebyscore：socre 升序 ；zrevrangebyscore ：score 降序）
  + *zscore*：获取指定 member 对应的 socre
  + *zrem*：删除指定 member 的元素

+ sorted set 底层通过有两种实现方式：`REDIS_ENCODING_ZIPLIST（压缩表）、REDIS_ENCODING_SKIPLIST（跳跃表 + 哈希表）`

  ![](image\sorted sort 实现.png)

  ```c
  /*
   · 结构选择：有序集 中添加的第一个元素 决定了 创建哪种结构
   
   	1、第一个元素符合以下条件时，创建 REDIS_ENCODING_ZIPLIST（压缩表）
   	  * 服务器属性 server.zset_max_ziplist_entries > 0（默认为 128）
   	  * 元素的 member 长度小于 server.zset_max_ziplist_value 值（默认值为 64MB）
   	
   	2、不满足上述条件时，创建 REDIS_ENCODING_SKIPLIST 有序集！
   
   · 结构转换
   	若初始创建但是 压缩表，则后续添加的元素不满足以下条件时，自动转换为 REDIS_ENCODING_SKIPLIST
   	  * 新添加的元素的 member 长度 >  server.zset_max_ziplist_value 值（默认值为 64MB）
   	  * 有序集中结点总数 > server.zset_max_ziplist_entries（默认为 128）
  */
  ```

+ **REDIS_ENCODING_SKIPLIST（跳跃表 + 哈希表）**

  + 当使用 REDIS_ENCODING_SKIPLIST 编码时，有序集合采用 zset 结构体

    ```c
    /*
     * 有序集：跳跃表+哈希表 共同实现
     */
    typedef struct zset {
    // 哈希表
    dict *dict;
    // 跳跃表
    zskiplist *zsl;
    } zset; 
    ```

  + zset 同时使用 哈希表 和 跳跃表 两个数据结构保存有序集合！

    元素的数据内容由一个 redisObject 结构表示，元素的 score 则是一个 double 浮点数，哈希表和跳跃表通过将指针共同指向这两个值来节约空间，而并没有将 元素复制两份！节约了内存空间！

    ![](image\sorted sort 实现_2.png)

    **哈希表中的 key 记录 member ；跳跃表通过 score 排序**

    ```c
    /*
     · 哈希表 将 member 作为键，socre 作为值，有序集可以在 O(1) 的时间复杂度内
     	* 检查给定的 member 是否在 有序集中
     	* 取出 member 对应的 socre 值（实现 zscore 指令）
     
     · 跳跃表：score
     	* 在 O(logN)期望时间、O(N)最坏时间内，根据 score 对 member 进行定位
     	* 范围性查找和处理操作，高效实现 zrank、zrange、zrangebyscore
     	* 跳跃表结点中有 前继指针和后继指针 ，高效实现 zrevrank、zrevrange、zrevrangebysocre
    */
    ```

+ 应用

  1. **排行榜**（最基础）

     视频网站需要对用户上传的视频做排行榜，榜单维护可能是多方面的：按照时间、按照播放量、按照获得的赞数等

     按照分数排名

     获取 TOPN

  2. **延时队列**

     使用 sorted sort ，使用 时间戳 作为 score，消息内容 作为 key 调用 zadd 来生产消息；消费者 用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理。

  3. **带权重的队列**
  
     普通消息时 设置 socre =1，重要消息时 设置  socre=2，然后工作线程可以选择按照 socre 升序或者倒序的方式 获取工作任务，让重要的任务先执行。

###### 其他类型

+ BitMap

  [bitmap 学习](https://www.jianshu.com/p/4c8e119f35db)

+ GEO

  [geo 应用](https://www.jianshu.com/p/0656e6ab538d)

+ HyperLogLog

  [HyperLogLog 非精确去重](https://baijiahao.baidu.com/s?id=1611726471431642966&wfr=spider&for=pc)

+ BloomFilter

  [布隆过滤器](https://mp.weixin.qq.com/s/1gsPVcQEnTH_96JWb360oA)

  [布隆过滤器 应用 与 原理介绍](https://mp.weixin.qq.com/s/BdwZViiAqnFhCde4ZsxwPg)

  [大数据量下的集合过滤：布隆过滤器](https://mp.weixin.qq.com/s/np5MW1DUwUlL--txLxrakA)

+ RediSearch

  [RediSearch 使用示例](https://www.jianshu.com/p/458319b4e47e)

  [RediSearch 性能对比](https://www.sohu.com/a/154690419_99937638)

  

  