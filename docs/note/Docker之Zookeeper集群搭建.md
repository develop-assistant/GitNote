# Docker之Zookeeper集群搭建

## 1. 集群搭建

**docker-compose.yml**

```yml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    volumes:
      - ./zookeeper/zoo1/data:/data
      - ./zookeeper/zoo1/datalog:/datalog

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181
    volumes:
      - ./zookeeper/zoo2/data:/data
      - ./zookeeper/zoo2/datalog:/datalog

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
    volumes:
      - ./zookeeper/zoo3/data:/data
      - ./zookeeper/zoo3/datalog:/datalog
```

**校验配置**

```shell
docker-compose -f docker-compose-zk.yml config -q
```

**启动集群**

```shell
docker-compose -f docker-compose-zk.yml up -d
```

**查看容器启动情况**

```shell
➜  docker docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                                  NAMES
ac21a50f548a        zookeeper           "/docker-entrypoint.…"   About a minute ago   Up 59 seconds       2888/tcp, 3888/tcp, 8080/tcp, 0.0.0.0:2182->2181/tcp   zoo2
901aa4eee887        zookeeper           "/docker-entrypoint.…"   About a minute ago   Up About a minute   2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, 8080/tcp   zoo1
5e4e87f4aea5        zookeeper           "/docker-entrypoint.…"   About a minute ago   Up 59 seconds       2888/tcp, 3888/tcp, 8080/tcp, 0.0.0.0:2183->2181/tcp   zoo3
```

**查看zookeeper集群状态**

```shell
➜  docker docker exec -it zoo1 /bin/sh
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
# exit

➜  docker docker exec -it zoo2 /bin/sh
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
# exit

➜  docker docker exec -it zoo3 /bin/sh
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
# exit
```

