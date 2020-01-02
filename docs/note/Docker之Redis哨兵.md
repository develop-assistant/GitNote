# Docker之Redis哨兵

## 1. 环境搭建

```
version: '3.7'
services:
  master:
    image: redis
    container_name: redis-master
    restart: always
    command: redis-server --port 6379 --requirepass 123456  --appendonly yes
    ports:
      - 6379:6379
    volumes:
      - ./redis/master/data:/data

  slave1:
    image: redis
    container_name: redis-slave-1
    restart: always
    command: redis-server --slaveof redis-master 6379 --port 6379  --requirepass 123456 --masterauth 123456  --appendonly yes
    ports:
      - 6380:6379
    volumes:
      - ./redis/slave1/data:/data


  slave2:
    image: redis
    container_name: redis-slave-2
    restart: always
    command: redis-server --slaveof redis-master 6379 --port 6379  --requirepass 123456 --masterauth 123456  --appendonly yes
    ports:
      - 6381:6379
    volumes:
      - ./redis/slave2/data:/data
```

**启动redis**

```
docker-compose -f docker-compose-redis.yml up -d
```

**查看启动日志**

```
docker logs -f 11f4bd6af2ba
```

**主从测试**

1、 在master中写入数据

```
➜  docker docker exec -it 9e4091def317 bash
root@9e4091def317:/data# redis-cli
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set name admin
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379>
```

2、 在slave中读取数据

```
➜  docker docker exec -it a8d8621693ba bash
root@a8d8621693ba:/data# redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> get name
"admin"
127.0.0.1:6379>
```

3、 在slave中写数据

```
➜  docker docker exec -it cec356c920f7 bash
root@cec356c920f7:/data# redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> set test admin
(error) READONLY You can't write against a read only replica.
127.0.0.1:6379>
```

4、查看映射持久化文件

```
➜  redis tree
.
├── master
│   └── data
│       ├── appendonly.aof
│       └── dump.rdb
├── slave1
│   └── data
│       ├── appendonly.aof
│       └── dump.rdb
└── slave2
    └── data
        ├── appendonly.aof
        └── dump.rdb
```

至此,redis一主二从完毕。(可以查看aof和rdb中内容)



## 2. 哨兵环境搭建