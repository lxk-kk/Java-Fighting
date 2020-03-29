#### 【Redis_哨兵】

##### 介绍

+ 主从复制的缺陷

  主从复制虽然能通过冗余数据，实现数据的热备份，但是，单独的主从复制依旧会出现单点故障问题，一旦主节点故障，则需要人工转移故障，更新并将从节点升级为主节点，这段时间内，服务器是无法对外提供服务的！

+ **Sentinel 哨兵**

  哨兵为实现 服务高可用而生，是 **Redis高可用的解决方案**！它是 一个特殊的 Redis 服务器，没有数据持久化功能，也不会执行 普通的数据操作命令！

  哨兵在主从复制的基础上，解决主节点单点故障的问题！有一个或者多个 Sentinel（实例）组成的 Sentinel系统 可以监视任意多个主服务器，以及这些主服务下的从服务器！当检测到 主服务器下线、宕机时，Sentinel 会主动将故障转移，并将该主服务器下的某个从服务器升级为主服务器，然后由新的主服务器代替宕机的服务器继续处理命令请求，实现服务高可用！

  > 注意：哨兵 + 主从部署：不能保证数据零丢失，只能保证 redis 集群高可用

+ Sentinel 哨兵 功能？

  1. 监控节点的存货状态：会定时发送 PING 命令，检测所有与 master 相关的 redis 实例的存活状态！
  2. 主动完成 下线节点的故障转移：自动在 旧master 中选取 某个slave 晋升为 新的master，代替旧的 master 对外提供服务！
  3. 主动更新 新的 master-slave 关系！
  4. 通知：将故障转移的结果通知给应用方？

##### 哨兵如何设置

+ **哨兵至少应该设置 ，并且最好是奇数个** ！

  哨兵是 Redis 的高可用机制，保证了 Redis 服务不会出现单点故障，如果哨兵只部署一个，哨兵本身就成了一个单点，不合理！如果哨兵部署 2 个，当 Redis 故障时，2 个哨兵之间相互选举，投递自己一票，根本就选举不出 leader，所以也是不可以的！所以哨兵个数应该大于 2 个，并且最好是奇数个！

##### 哨兵 启动 与 初始化

###### 启动

```shell
# 两种启动命令，效果完全相同
# 启动 1
> redis-sentinel /your_path/sentinel.conf
# 启动 2
> redis-server /your_path/sentinel.conf --sentinel
```

###### 初始化

```c
/*
 · 主流程初始化
 	哨兵初始化 与 普通服务器初始化过程 并不安全相同：它不需要做数据持久化，因此不会载入 RDB文件 或者 AOF 文件；其次，哨兵不提供数据服务，它拥有自己独有的命令集，因此它需要更改可执行命令！
*/
int main(){
        // ...
    // 检测 是否以 sentinel 模式启动
    server.sentinel_mode = checkForSentinelMode(argc,argv);
    // ...
    if(server.sentinel_model){
        
        // 初始化配置文件，将端口设置为 26379                
        initSentinelConfig();
        // 更改哨兵可执行命令，并初始化哨兵实例
        // 哨兵中只能执行个别的命令：ping、sentinel、subscribe、publish、info等                
        initSentinel();
        // ...                
    }
    // ...
    // 解析配置文件，进行初始化    
    sentinelHandleConfiguration();
    // ...
    // 随机生成一个 40 字节的哨兵 运行ID，打印启动日志    
    sentinelIsRunning();
    // ...                                    
}
```

##### 哨兵 建立连接并探测

```c
serverCron(){
    if(server.sentinel_mode){
        sentinelTimer();
    }
}
```

+ 初始化完成后，哨兵通过 时间事件 开始工作！

+ 哨兵每次执行 serverCron 时，都会调用 sentinelTimer() 函数，该函数会建立连接并且定时发送心跳包 采集信息！该函数主要功能如下：

  1. 建立网络连接（命令连接 与 订阅连接）
  2. 信息采集 与 探测

  3. 检测服务主观下线
  4. 检测服务客观下线

###### 建立 master 网络连接

+ 上述初始化过程之后，sentinel 就会创建 与 所监视的master 的网络连接，包括 **命令连接 与 订阅连接** !

  1. 命令连接：专门用于向 master 发送命令请求，并接收命令回复！
  2. 订阅链接：专门用于订阅 master的 `__sentinel__:hello` 频道！

+ 为什么需要有两个连接？

  1. 订阅连接：

     **因为 sentinel 需要通过接收 服务器的 `__sentinel__:hello`频道信息，来发现其他的 sentinel ，所以才需要建立订阅连接！**在 redis 的发布订阅功能中，被发送的信息不会保存在 redis服务器中，如果信息发送时，需要接收消息的客户端不在线，则这个客户端将会丢失这条信息，因此，为了不丢失 `__sentinel__:hello` 频道中 其他 sentinel 的信息，需要建立 订阅连接！

  2. 命令连接

     除了订阅频道之外，sentinel 还需要向 master 发送命令 与 master 通信，所以 Sentinel 还必须创建命令连接！

######  master 信息采集

1. 获取 主服务器信息（**INFO 命令**）

   sentinel *默认以 10s 每次 的频率*，通过命令连接*向被监视的 master 发送 INFO 命令*，并通过 命令回复 采集主服务器信息！ 

   ```c
   /*
    · Sentinel 通过 INFO 命令获取以下信息：
   	1. 主服务器自身的信息：runid、role（角色 master / slave）
   	2. master 下所有的 slave 信息：ip、port、state（状态：online）、offset（复制偏移量）、lag
   
    · 所以，无需向 Sentinel 提供从服务器的信息，Sentinel 能通过 master 自动发现所有 slave！
    	
   */
   ```

2. 获取 从服务器信息（**INFO 命令**）

   Sentinel 通过 master 发现它的 新salve 之后，便会*创建 sentinel 与 salve 的 命令连接与 订阅连接* ！

   在*创建命令连接* 之后，默认会*以 10s 每次 的频率*，通过命令连接 *向 salve 发送 INFO 命令*，采集 slave 信息！

   ```c
   /*
    · info 命令获取以下信息：
   		runid ：salve 的运行 ID
   		role ：角色
   		master_host ：master 的 ip
   		master_port ：master 的 port
   		master_link_status ：master-slave 的连接状态
   		slave_priority ：slave 的优先级
   		slave_repl_offset ：salve 的复制偏移量
   */
   ```

3. 向 master、slave 发送 信息（**`PUBLISH __sentinel__:hello`命令**）

   默认情况下，sentinel 会*以 2s 每次的频率*，通过 命令连接*向所有被监视的 master、slave 的 `__sentinel__:hello` 频道发送  PUBLISH 命令*：

   ```shell
   > PUBLISH __senitnel__hello: "<s_ip>,<s_port>,<s_runid>,<s_epoch>,<m_name>,<m_ip>,<m_port>,<m_epoch>"
   ```

   ```c
   /*
    · <s_ip>,<s_port>,<s_runid>,<s_epoch> 
    	表示 sentinel 当前信息，分别为：ip、port、运行id、当前配置纪元（用于选举 和 主从切换）
    
    · <m_name>,<m_ip>,<m_port>,<m_epoch>
    	如果 sentinel 正在监视的是 master，则就是该 master 的信息
    	如果 sentinel 正在监视的是 slave，则就是该 slave 的 master 的信息
    	
    	分别为：master名称、master ip、master port、master的配置纪元（用于选举 和 主从切换）
   */
   ```

4. 订阅 master、slave 的频道信息（**`SUBSCRIBE __sentinel__:hello`命令**）

   当 sentinel 与一个 master / salve 建立起 订阅连接，sentinel 就会通过 订阅连接，发送 SUBSCRIBE 命令订阅 `__sentinel__:hello`频道的消息

   > `SUBSCRIBE __sentinel__:hello`
   >
   > *sentinel 会一直订阅 `__sentinel__:hello` 频道的信息，直到与 服务器的 连接断开为止*

   ```c
   /*
    · 综上
    	每个 sentinel 既  通过 命令连接 向服务器的 __sentinel__:hello 频道发送（publish）消息
    	每个 sentinel 又  通过 订阅连接 从服务器的 __sentinel__:hello 频道订阅（subscribe）消息
   */
   ```

   ![](image\sentinel 发布订阅消息.png)

5. 发现并连接 其他sentinel

   + 对于监视同一个 服务器的多个 sentinel 而言，*一个 sentinel 发送到 `__sentinel__:hello`频道的消息，会被其他 sentinel 订阅到*，这些消息用于更新其他 sentinel 对 发送者（sentinel） 以及 被监视服务器（master / slave） 的认知！

     ```c
     /*
      · 当一个 sentinel 从 __sentinel__:hello 频道接收到一条信息后，从中发现其他 sentinel
      	1）如果信息中 sentinel runid 与自身的 runid 相同，则说明是自身发送的，信息被丢弃！
        2）如果信息中 sentinel runid 与自身的不同，说明是监视同一个服务器的其他 sentinel 发送的，于是，记录这个 sentinel 的信息！ 
     */
     ```

   + 当 sentinel 发现其他 sentinel 之后，不仅会记录该 sentinel 的信息，还会与该 sentinel 建立 *命令连接* ，最终 监视同一个 master 的所有 sentinel 将形成相互连接的网络 —— 开始 sentinel 之间的通信！

   + 使用 命令连接 相连的各个 sentinel 可以向其他的 sentinel 发送命令请求，达到信息交换的目的！

   + sentinel 之间不会创建 订阅连接

     **因为 sentinel 需要通过接收 服务器的 `__sentinel__:hello`频道信息，来发现其他的 sentinel ，所以才需要建立订阅连接，而相互已知的 sentinel 只需要使用 命令连接 进行通信就足够！*

###### 检测主观下线

+ 默认情况下，sentinel 会以 *每秒1次 的频率*，*向所有与它建立连接的 实例（master、slave、sentinel）发送 PING 命令*，并根据命令回复，判断其是否在线！

+ PING 的回复有两种情况：

  1. *有效恢复* ：*+PONG、-LOADING、-MASTERDOWN* 中的一种
  2. 无效回复 ：上述回复之外的 或者 在指定时限内没有返回任何回复的

+ 判定主观下线：

  **在 down-after-milliseconds（毫秒）内，一个实例始终是无效回复，则当前 sentinel 判定该实例为 主观下线状态！**

  down-after-milliseconds 属性不仅用于判断 master，还用于判断与 master 相关的所有 slave、sentinel 实例！

  ```shell
  # 示例：
  sentinel monitor master 127.0.0.1 6379 2
  sentinel down-after-milliseconds master 5000
  # 5000ms 不仅是 master 进入主观下线状态的标准，还是其 所有 slave、sentinel 主观下线的标准！ 
  ```

+ 注意：*多个 sentinel 设置的主观下线时限可能不同！*

  每个 sentinel 中都有自己的 sentinel down-after-milliseconds 属性配置，并且可能各自不相同，对于 服务器 是否处于下线状态，各抒己见，所以实例是主观的下线状态！

###### 检测客观下线

+ 当 sentinel 判定某个 master 为主观下线时，需要向监视该 master 的其他 sentinel 获取看法，如果大多数 sentinel 都认为该 master 为主观下线，*则 sentinel 会将该 master 判定为 客观下线* 状态，*并执行 故障转移操作* ！

+ 询问其他 sentinel：**SENTINEL is-master-down-by-addr 命令**

  ```shell
  > SENTINEL is-master-down-by-addr <m_ip> <m_port> <currnet_epoch> *
  # 最后一个 参数为 * ：代表该命令用于检测 master 客观下线状态！
  ```

  > *【补充】SENTINEL is-master-down-by-addr 指令的两个作用，区别在于，最后一项参数！*
  >
  > ```shell
  > > SENTINEL is-master-down-by-addr <m_ip> <m_port> <currnet_epoch> ( < runid > 或者 * )
  > ```
  >
  > ```c
  > /*
  >  	<m_ip> ：master 的 ip
  >  	<m_port> ：master 的 port
  >  	<currnet_epoch> ：sentinel 当前的配置纪元（用于选举 leader）
  >  	
  >  	最后一项参数 ：可以是 * 或者 sentinel的runid
  >  		* 		代表：该命令用于检测 master 客观下线状态！
  >  		runid 	代表：该命令用于选举 leader
  > */
  > ```

+ 接收其他 sentinel 的 SENTINEL is-master-down-by-addr

  当 sentinel（目标） 接收到其他 sentinel（源）发送的 SENTINEL is-master-down-by-addr 命令时，会根据该指令的最后一项参数，判断该指令的意图（询问master 下线？选举 leader？）（此处，默认是 询问 master 的状态！）

  当前 sentinel 从指令中获取所要 咨询的 master 信息，检查该 master 是否下线，并将项结果 包含在一个 Multi Bulk 中作为回复！

  ```c
  /*
   · 将如下参数包含在 multi bulk 内
   	<down_state>	：目标 sentinel 对 master 状态的判断结果（0：在线，1：下线）
   	<leader_runid>  ：*（用于选举时使用，在回复 master 状态时，应为 *）
   	<leader_epoch>  ：0（用于选举时使用，在回复 master 状态时，应为 0）
  */
  ```

+ 接收其他 sentinel 对 SENTINEL is-master-down-by-adrr 的回复

  当前 sentinel 统计其他 sentinel 对指定 master 的状态判断，当达到判定所需数量时（**quorum 配置**），将 master 判定为 客观下线状态！

  > **sentinel monitor master < m_ip > < m_port > < quorum >**
  >
  > ```shell
  > # 例：
  > sentinel monitor master 127.0.0.1 6379 3
  > # 本 sentinel 监视 127.0.0.1:6379 这个 master
  > # 当 监视这个 master 的所有 sentinel（包括自己）中有 3 个，认为这个 master 是下线状态，则本sentinel 判定这个 master 为客观下线状态！
  > ```

+ 注意

  1、判定 主观下线 会针对所有 实例，而判定 客观下线 只针对 master！（只有 master 才会主观下线）

  2、不同的 sentinel 对同一个 master 的客观下线判定可能不相同，因为 配置时 quorum 可能不同！

##### 哨兵 故障转移（主从切换）

+ 当 Redis 哨兵方案中 master 处于客观下线状态时，为了保证 Redis 高可用性，此时需要将 故障转移 进行主从切换。将 master 中的某个 slave 升级为 master，令其他 slave 复制这个 新的master！

1. 当一个哨兵 发现某台 master 处于客观下线状态时，会首先将 *切换状态 更新为 SENTINEL_FAILOVER_STATE_WAIT_START* ：发送选举命令，要求 其他 哨兵为自己投票！进入 Leader 选举！

   `详见下一节《哨兵 Leader》`

   ```shell
   > SENTINEL is-master-down-by-addr <m_ip> <m_port> <s_epcoh> <s_runid>
   # s_epoch ：哨兵当前的 配置纪元 ：每一轮 选举后都会 +1
   # s_runid ：哨兵的 运行ID ：要求其他 sentinel 选择自己为 leader
   ```

   选出 leader 之后，剩下的工作将交由 leader 完成！

2. Leader 首先会将  *切换状态 更改为 SENTINEL_FAILOVER_STATE_SELECT_SLAVE* ：从下线的 master 的 slave 中选择一个作为新的 master。

   ```c
   /*
    · 选择新 master 的规则：
    	1）如果该 slave 处于下线状态，则 pass
    	2）如果该 slave 5s 之内没有有效回复 ping命令，或者 与主服务器断开时间过长，则 pass
    	3）如果该 slave_priority = 0，则 pass
    	  配置属性 slave_priority ：值为正整数，值越小优先级越高（为 0 时，表示不能被选为 master）
    	
    	--------------------------------------------------------------------------
    	剩下的 slave 保存的数据都是最新的
    	--------------------------------------------------------------------------
    	
   	4）在剩余的 slave 中比较优先级，优先级高的被选中
    	5）优先级相同的，则选择 复制偏移量最大的
    	6）优先级相同，切复制偏移量都最大的，选择其中 运行ID 最小的！
   */
   ```

3. 选中从服务器后，将 *切换状态 更改为 SENTINEL_FAILOVER_STATE_SLAVEOF_NOOE* ：将选择的 redis slave 切换为 redis master，即向哨兵发送如下命令：**SLAVEOF no one**

   ```shell
   # 开启一个事务
   MULTI
   # 关闭从服务器的复制功能，将其转换为一个主服务器！
   SLAVEOF NO ONE
   # 将 redis.conf 文件重写（会根据当前运行中的配置重写原来的配置）
   CONFIG REWRITE
   # 关闭连接到该服务器的客户端（关闭之后客户端会重连，重连时会重新获取 Redis master 的地址）
   CLIENT KILL TYPE normal
   # 提交事务
   EXEC
   ```

   如果本次切换超时，哨兵将放弃本次切换，再重第一步开始重新执行切换！

4. 切换完成后，将*切换状态 更新为 SENTINEL_FAILOVER_STATE_RECONF_SLAVES* ：leader 向剩余的 slave 发送命令，修改 slave 的复制目标：**SLAVEOF new_master_ip new_master_port**

   ```shell
   # 开启一个事务
   MULTI
   # 令该服务器 复制 新的主服务器
   SLAVEOF new_master_ip new_master_port
   # 将 redis.conf 配置文件重写（会根据当前运行中的配置 重写 原来的配置）
   CONFIG REWRITE
   # 关闭连接到该服务器的客户端（关闭连接之后客户端会重连，重连时重新获取 redis master 地址）
   CLIENT KILL TYPE noromal
   # 执行事务
   EXEC
   ```

5. 如果所有从服务器更新完毕，则 *切换状态 更新为 SENTINEL_FAILOVER_STATE_UPDATE_CONFIG* ：将 旧的master  修改为 slave，当 其上线后，leader 会向其发送 SLAVEOF 命令，复制新的 master

6. 完成上述工作，*切换状态 更新为 SENTINEL_FAILOVER_STATE_NONE* ：表示主从切换完成

   切换状态过程：

![](image\主从切换状态.png)

##### 哨兵 Leader

+ 当一个 master 被判断为 客观下线时，监视这个 master 的各个 sentinel 会进行协商，选举出一个领头 sentinel，并由该 leader 对下线 master 执行故障转移！

+ **leader 选举过程**

  1. 每个`发现主服务器进入客观下线`的 sentinel`（源sentinel）` 都会发送 选举命令**SENTINEL is-master-down-by-addr（末尾参数为 发送者 runid）**，`要求其他sentinel（目标sentinel）`将自己设置为 局部 leader。

     *如果，源sentinel 自身没有为其他 sentinel 投过票，则它会首先给自己投一票！*

     > **SENTINEL is-master-down-by-addr（末尾参数为 发送者 runid）**

  2. `目标sentinel `收到命令后，采取*先来先设置* 的规则，将最早发送命令的 `源sentinel` 设置为自己的 leader（可能是自己），而后续的 `源sentinel` ，则被拒绝。

     而后，向 `源sentinel` 进行回复，回复内容为 `局部leader` 的信息。

     ```c
     /*
      · 目标sentienl 会将如下参数包含在 multi bulk 内，回复给 源sentinel
       
      	<down_state>	：（判断 master 客观下线时使用）
      	<leader_runid>  ： 目标 sentinel 的 局部leader 的 运行ID
      	<leader_epoch>  ： 目标 sentinel 的 局部leader 的 配置纪元 	
     */
     ```

  3. `源sentienl` 收到 `目标sentinel` 回复后，会检查回复中的 leader_epoch 和 leader_runid 是否和自己的一致。若一致，则表明`目标sentinel`将 自己 设置成了局部leader

  4. 如果有某个 sentinel 被半数以上的 sentinel 设置成了 局部leader，那么这个 sentinel 会成为 Leader Sentinel！

  5. 如果在给定时限内，没有一个真正的 sentinel 成为 领头Sentinel ，那么各个 Sentinel 将在一段时间之后再次进行选举，直到选出 Sentinel为止！

+ 注意：

  1. 监视 master 的所有 sentinel 都有资格称为 leader。
  2. 每轮 leader 选举之后，不论是否选举成功，所有 sentinel 的*配置纪元（configuration epoch）都会自增* 一次。
  3. 在一个 配置纪元中，所有 sentinel 都有一次将某个 sentinel 设置为 局部leader 的机会，并且*局部 leader 一旦设置，在这个 配置纪元中就不能再更改*！
  4. 领头 sentinel 的产生需要半数以上的 sentinel 的支持，并且*每个 sentinel 在每个配置纪元中只能设置一次局部leader*，所以在一个配置纪元中，只会出现一个 领头sentinel

##### Sentinel 中存在的问题

1. 两个 sentinel 中的一个 故障了，则另一个 sentinel 的选举leader永远失败
2. sentinel 不能识别 slave 中的 127.0.0.1 地址
3. 修改 sentinel 模式下的实例角色时，也需要修改 sentinel 中和该实例相关的配置
4. 被 sentinel 监控的 slave ，使用 slaveof 命令无效？
5. sentinel 无法将 slave 当作 master 作为监控对象
6. sentinel 故障迁移中，slave 重配超时后，sentinel 向其他 slave 发送的 slaveof 命令被丢弃！

##### 总结

1. 主从切换完成之后，客户端和其他哨兵如何知道现在提供服务的 master 是哪一个呢？

   > **通过 subscribe_sentinel__:hello 频道，知道当前提供服务器 master 的 ip **

2. 执行切换的哨兵发生了故障，切换动作是否会由其他哨兵继续完成呢？

   > **执行切换的哨兵发生故障后，剩余的哨兵会重新选择出 leader，并且重新执行切换工作**

3. 故障 master 恢复之后，会继续作为 master 提供服务还是会作为 slave 提供服务？

   > **redis 主从切换完成之后，当故障 redis master 恢复之后，会作为新的 master 的一个 slave 提供服务**