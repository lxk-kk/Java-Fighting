#### 【Redis_集群】

#### 集群

##### 集群一致性hash算法

#### 集群高可用如何保证：

1. Redis sentinel 着眼于 主从高可用，在 master 宕机后，能自动将其从节点升级为新的master，继续提供服务！
2. Redis Cluster 着眼于 集群扩展性，在当个 redis 内存不足时，可以使用 Cluster 进行分片存储！