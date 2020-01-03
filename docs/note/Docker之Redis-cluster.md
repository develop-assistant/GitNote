# Docker之Redis-cluster

## 1. 环境搭建

**规划网络配置**

首先创建redis集群的网络

docker network create --subnet=10.0.0.0/16 redisnet

| ip       | port |
| -------- | ---- |
| 10.0.0.2 | 7001 |
| 10.0.0.3 | 7002 |
| 10.0.0.4 | 7003 |
| 10.0.0.5 | 7004 |
| 10.0.0.6 | 7005 |
| 10.0.0.7 | 7006 |



**docker-compose-redis-cluster.yml**

```
version: '3.7'

services:
  redis7001:
    image: 'redis'
    container_name: redis7001
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7001/data:/data
    ports:
      - "7001:6379"
      - "17001:16379"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    networks:
      redisnet:
        ipv4_address: 10.0.0.2

  redis7002:
    image: 'redis'
    container_name: redis7002
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7002/data:/data
    ports:
      - "7002:6379"
      - "17002:16379"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    networks:
      redisnet:
        ipv4_address: 10.0.0.3

  redis7003:
    image: 'redis'
    container_name: redis7003
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7003/data:/data
    ports:
      - "7003:6379"
      - "17003:16379"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    networks:
      redisnet:
        ipv4_address: 10.0.0.4

  redis7004:
    image: 'redis'
    container_name: redis7004
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7004/data:/data
    ports:
      - "7004:6379"
      - "17004:16379"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    networks:
      redisnet:
        ipv4_address: 10.0.0.5

  redis7005:
    image: 'redis'
    container_name: redis7005
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7005/data:/data
    ports:
      - "7005:6379"
      - "17005:16379"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    networks:
      redisnet:
        ipv4_address: 10.0.0.6

  redis7006:
    image: 'redis'
    container_name: redis7006
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7006/data:/data
    ports:
      - "7006:6379"
      - "17006:16379"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    networks:
      redisnet:
        ipv4_address: 10.0.0.7

networks:
  redisnet:
    driver: bridge
    ipam:
      config:
        - subnet: 10.0.0.0/16

```



**配置文件**

redis.conf完整配置文件见官网。

<a href="../assets/redis.conf" target="_blank">附件</a>

这里我们用自己的配置文件redis.conf

```conf
port 6379
# 开启集群
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 5000
pidfile "/var/run/redis_6379.pid"
dir "/data"
logfile "/data/redis-6379.log"
requirepass "123456"
masterauth 123456
appendonly yes
```

> 重要:  配置文件映射，docker镜像redis 默认无配置文件。



**启动容器**

```
docker docker-compose -f docker-compose-redis-cluster.yml up -d
```

**配置集群**(不同版本详见[官方文档](https://redis.io/topics/cluster-tutorial))

```
docker exec -it redis7001 redis-cli -p 6379 -a 123456 --cluster create 10.0.0.2:6379 10.0.0.3:6379 10.0.0.4:6379 10.0.0.5:6379 10.0.0.6:6379 10.0.0.7:6379 --cluster-replicas 1
```

结果如下

```
➜  docker docker exec -it redis7001 redis-cli -p 6379 -a 123456 --cluster create 10.0.0.2:6379 10.0.0.3:6379 10.0.0.4:6379 10.0.0.5:6379 10.0.0.6:6379 10.0.0.7:6379 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 10.0.0.6:6379 to 10.0.0.2:6379
Adding replica 10.0.0.7:6379 to 10.0.0.3:6379
Adding replica 10.0.0.5:6379 to 10.0.0.4:6379
M: 4579ff67a2ae08275e66ac72eb5cfdbf6bb6b697 10.0.0.2:6379
   slots:[0-5460] (5461 slots) master
M: 39d27f77e0e8102ab4d9d46cc9b26ba0d881247d 10.0.0.3:6379
   slots:[5461-10922] (5462 slots) master
M: 659b10c76b723da9caa33b51ad2f087739d481d5 10.0.0.4:6379
   slots:[10923-16383] (5461 slots) master
S: 08d29202d75af825ebda488f254a29e2b51f8699 10.0.0.5:6379
   replicates 659b10c76b723da9caa33b51ad2f087739d481d5
S: 0a46bc61364ac7c7a1e42d907b79dfdb72b71bd7 10.0.0.6:6379
   replicates 4579ff67a2ae08275e66ac72eb5cfdbf6bb6b697
S: 8e6b886399e51e6ca8b096ac63130642e56900a6 10.0.0.7:6379
   replicates 39d27f77e0e8102ab4d9d46cc9b26ba0d881247d
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 10.0.0.2:6379)
M: 4579ff67a2ae08275e66ac72eb5cfdbf6bb6b697 10.0.0.2:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 0a46bc61364ac7c7a1e42d907b79dfdb72b71bd7 10.0.0.6:6379
   slots: (0 slots) slave
   replicates 4579ff67a2ae08275e66ac72eb5cfdbf6bb6b697
S: 08d29202d75af825ebda488f254a29e2b51f8699 10.0.0.5:6379
   slots: (0 slots) slave
   replicates 659b10c76b723da9caa33b51ad2f087739d481d5
M: 659b10c76b723da9caa33b51ad2f087739d481d5 10.0.0.4:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 8e6b886399e51e6ca8b096ac63130642e56900a6 10.0.0.7:6379
   slots: (0 slots) slave
   replicates 39d27f77e0e8102ab4d9d46cc9b26ba0d881247d
M: 39d27f77e0e8102ab4d9d46cc9b26ba0d881247d 10.0.0.3:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



## 2. 容灾演练

