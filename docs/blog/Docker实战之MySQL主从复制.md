# 前言

曾几何时，看着高大上的架构和各位前辈高超的炫技，有没有怦然心动，也想一窥究竟？每当面试的时候，拿着单应用的架构，吹者分库分表的牛X，有没有心里慌的一批？

其实很多时候，我们所缺少的只是对高大上的技术的演练。没有相关的业务需求，没有集群环境，然后便只是Google几篇博文，看下原理，便算是了解了。然而真的明白了吗？众多的复制粘贴中，那篇文章才对我们有用，哪些又是以讹传讹？

所幸容器技术的快速发展，让各种技术的模拟成为现实。接下来Docker相关的一系列文章，将以实战为主，帮助大家快速搭建测试和演练环境。

# Docker文件编排

由于是测试演为了练用，这里用docker-compose进行配置文件的编排，实际的集群环境中并不是这么部署的。

1. 编排docker-compose-mysql-cluster.yml,安装master和slave节点

```yaml
version: '3'
services:
  mysql-master:
    image: mysql:5.7
    container_name: mysql-master
    environment:
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - "3307:3306"
    volumes:
      - "./mysql/master/my.cnf:/etc/my.cnf"
      - "./mysql/master/data:/var/lib/mysql"
    links:
      - mysql-slave

  mysql-slave:
    image: mysql:5.7
    container_name: mysql-slave
    environment:
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - "3308:3306"
    volumes:
      - "./mysql/slave/my.cnf:/etc/my.cnf"
      - "./mysql/slave/data:/var/lib/mysql"

```

2. 配置master配置文件my.cnf

```
[mysqld]
# [必须]启用二进制日志
log-bin=mysql-bin 
# [必须]服务器唯一ID，默认是1，一般取IP最后一段  
server-id=1
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
```

3. 配置slave配置文件my.cnf
 
```
[mysqld]
# [必须]服务器唯一ID，默认是1，一般取IP最后一段  
server-id=2
```

4. 启动docker-compose，创建docker镜像文件

```docker
docker-compose -f docker-compose-mysql-cluster.yml up -d
```

`docker ps`查看进程，可以看到2个实例已启动。

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
1f4ad96b4d5e        mysql:5.7           "docker-entrypoint.s…"   48 seconds ago      Up 46 seconds       33060/tcp, 0.0.0.0:3307->3306/tcp   mysql-master
8e2214aacc43        mysql:5.7           "docker-entrypoint.s…"   49 seconds ago      Up 47 seconds       33060/tcp, 0.0.0.0:3308->3306/tcp   mysql-slave
```


# 配置主从复制

# 结果验证