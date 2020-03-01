# 概述

接上一篇[Docker实战之MySQL主从复制](https://mp.weixin.qq.com/s/3FbY6jT-PdgUHsRwHBSWBw), 这里是Docker实战的第二篇，主要进行Redis-Cluster集群环境的快速搭建。Redis作为基于键值对的NoSQL数据库，具有高性能、丰富的数据结构、持久化、高可用、分布式等特性，同时Redis本身非常稳定，已经得到业界的广泛认可和使用。

在Redis中，集群的解决方案有三种

1. 主从复制
2. 哨兵机制
3. Cluster

Redis Cluster是Redis的分布式解决方案，在 3.0 版本正式推出。

# 集群方案的对比

**1. 主从复制**

同Mysql主从复制的原因一样，Redis虽然读取写入的速度都特别快，但是也会产生读压力特别大的情况。为了分担读压力，Redis支持主从复制，读写分离。一个Master可以有多个Slaves。

![](https://gitee.com/idea360/oss/raw/master/images/redis-master-slave.jpg)

优点

- 数据备份
- 读写分离，提高服务器性能

缺点

- 不能自动故障恢复,RedisHA系统（需要开发）
- 无法实现动态扩容

**2. 哨兵机制**

Redis Sentinel是社区版本推出的原生`高可用`解决方案，其部署架构主要包括两部分：Redis Sentinel集群和Redis数据集群。

其中Redis Sentinel集群是由若干Sentinel节点组成的分布式集群，可以实现故障发现、故障自动转移、配置中心和客户端通知。Redis Sentinel的节点数量要满足2n+1（n>=1）的奇数个。

![](https://gitee.com/idea360/oss/raw/master/images/redis-sentinel.png)

优点

- 自动化故障恢复

缺点

- Redis 数据节点中 slave 节点作为备份节点不提供服务
- 无法实现动态扩容


**3. Redis-Cluster**

Redis Cluster是社区版推出的Redis分布式集群解决方案，主要解决Redis分布式方面的需求，比如，当遇到单机内存，并发和流量等瓶颈的时候，Redis Cluster能起到很好的负载均衡的目的。

Redis Cluster着眼于`提高并发量`。

群集至少需要3主3从，且每个实例使用不同的配置文件。

在redis-cluster架构中，`redis-master节点一般用于接收读写，而redis-slave节点则一般只用于备份`， 其与对应的master拥有相同的slot集合，若某个redis-master意外失效，则再将其对应的slave进行升级为临时redis-master。 

在redis的官方文档中，对redis-cluster架构上，有这样的说明：在cluster架构下，默认的，一般redis-master用于接收读写，而redis-slave则用于备份，`当有请求是在向slave发起时，会直接重定向到对应key所在的master来处理`。 但如果不介意读取的是redis-cluster中有可能过期的数据并且对写请求不感兴趣时，则亦可通过`readonly`命令，将slave设置成可读，然后通过slave获取相关的key，达到读写分离。具体可以参阅redis[官方文档](https://redis.io/commands/readonly)等相关内容

![](https://gitee.com/idea360/oss/raw/master/images/redis-cluster.jpg)

优点

- 解决分布式负载均衡的问题。具体解决方案是分片/虚拟槽slot。
- 可实现动态扩容
- P2P模式，无中心化

缺点

- 为了性能提升，客户端需要缓存路由表信息
- Slave在集群中充当“冷备”，不能缓解读压力

# 网络规划

这里没有搭建虚拟机环境，全部在本地部署。本机的ip为 `192.168.124.5`

| ip            | port |
| ------------- | ---- |
| 192.168.124.5 | 7001 |
| 192.168.124.5 | 7002 |
| 192.168.124.5 | 7003 |
| 192.168.124.5 | 7004 |
| 192.168.124.5 | 7005 |
| 192.168.124.5 | 7006 |

# Redis配置文件

在docker环境中，配置文件映射宿主机的时候，(宿主机)必须有配置文件。[附件](https://raw.githubusercontent.com/antirez/redis/5.0.7/redis.conf)在这里。大家可以根据自己的需求定制配置文件。

下边是我的配置文件 `redis-cluster.tmpl`

```text
# redis端口
port ${PORT}
# 关闭保护模式
protected-mode no
# 开启集群
cluster-enabled yes
# 集群节点配置
cluster-config-file nodes.conf
# 超时
cluster-node-timeout 5000
# 集群节点IP host模式为宿主机IP
cluster-announce-ip 192.168.124.5
# 集群节点端口 7001 - 7006
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
# 开启 appendonly 备份模式
appendonly yes
# 每秒钟备份
appendfsync everysec
# 对aof文件进行压缩时，是否执行同步操作
no-appendfsync-on-rewrite no
# 当目前aof文件大小超过上一次重写时的aof文件大小的100%时会再次进行重写
auto-aof-rewrite-percentage 100
# 重写前AOF文件的大小最小值 默认 64mb
auto-aof-rewrite-min-size 64mb
```

由于节点IP相同，只有端口上的差别，现在通过脚本 `redis-cluster-config.sh` 批量生成配置文件

```bash
for port in `seq 7001 7006`; do \
  mkdir -p ./redis-cluster/${port}/conf \
  && PORT=${port} envsubst < ./redis-cluster.tmpl > ./redis-cluster/${port}/conf/redis.conf \
  && mkdir -p ./redis-cluster/${port}/data; \
done
```

生成的配置文件如下图

![](https://gitee.com/idea360/oss/raw/master/images/redis-cluster-config-tree.png)


# Docker环境搭建

这里还是通过docker-compose进行测试环境的docker编排。


# 容灾演练

# Redis持久化

# Redis主从同步

