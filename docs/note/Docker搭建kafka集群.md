# Docker搭建kafka集群

```yml
version: '3'
services:
  zk1:
    image: confluentinc/cp-zookeeper:latest
    hostname: zk1
    container_name: zk1
    restart: always
    ports:
      - "12181:2181"
    volumes:
      - "./zkcluster/zk1/data:/data"
      - "./zkcluster/zk1/datalog:/datalog"
    environment:
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
      ZOOKEEPER_SERVERS: zk1:12888:13888;zk2:22888:23888;zk3:32888:33888

  zk2:
    image: confluentinc/cp-zookeeper:latest
    hostname: zk2
    container_name: zk2
    restart: always
    ports:
      - "22181:2181"
    volumes:
      - "./zkcluster/zk2/data:/data"
      - "./zkcluster/zk2/datalog:/datalog"
    environment:
      ZOOKEEPER_SERVER_ID: 2
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
      ZOOKEEPER_SERVERS: zk1:12888:13888;zk2:22888:23888;zk3:32888:33888

  zk3:
    image: confluentinc/cp-zookeeper:latest
    hostname: zk3
    container_name: zk3
    restart: always
    ports:
      - "32181:2181"
    volumes:
      - "./zkcluster/zk3/data:/data"
      - "./zkcluster/zk3/datalog:/datalog"
    environment:
      ZOOKEEPER_SERVER_ID: 3
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
      ZOOKEEPER_SERVERS: zk1:12888:13888;zk2:22888:23888;zk3:32888:33888

  kafka1:
    image: confluentinc/cp-kafka:latest
    hostname: kafka1
    container_name: kafka1
    restart: always
    depends_on:
      - zk1
      - zk2
      - zk3
    environment:
      KAFKA_ADVERTISED_HOST_NAME: kafka1
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zk1:2181,zk2:2181,zk3:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
    volumes:
      - ./kfkluster/kafka1/logs:/kafka

  kafka2:
    image: confluentinc/cp-kafka:latest
    hostname: kafka2
    container_name: kafka2
    restart: always
    depends_on:
      - zk1
      - zk2
      - zk3
    environment:
      KAFKA_ADVERTISED_HOST_NAME: kafka2
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zk1:2181,zk2:2181,zk3:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9092
    volumes:
      - ./kfkluster/kafka2/logs:/kafka

  kafka3:
    image: confluentinc/cp-kafka:latest
    hostname: kafka3
    container_name: kafka3
    restart: always
    depends_on:
      - zk1
      - zk2
      - zk3
    environment:
      KAFKA_ADVERTISED_HOST_NAME: kafka3
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zk1:2181,zk2:2181,zk3:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9092
    volumes:
      - ./kfkluster/kafka3/logs:/kafka

  kafka_manager:
    image: hlebalbau/kafka-manager:latest
    hostname: kafka_manager
    container_name: kafka_manager
    restart: always
    ports:
      - "9000:9000"
    environment:
      ZK_HOSTS: "zk1:2181,zk2:2181,zk3:2181"
      APPLICATION_SECRET: "random-secret"
      KAFKA_MANAGER_AUTH_ENABLED: "true"
      KAFKA_MANAGER_USERNAME: admin
      KAFKA_MANAGER_PASSWORD: admin
    command: -Dpidfile.path=/dev/null
```

