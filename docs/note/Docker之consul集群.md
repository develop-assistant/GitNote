docker-compose-consul.yml

```yml
version: '3'
services:
  consul-server1:
    image: consul:latest
    hostname: "consul-server1"
    ports:
      - "8500:8500"
      - "53"
    volumes:
      - ./consul/data1:/consul/data
    command: "agent -server -bootstrap-expect 3 -ui -disable-host-node-id -client 0.0.0.0"
  consul-server2:
    image: consul:latest
    hostname: "consul-server2"
    ports:
      - "8501:8500"
      - "53"
    volumes:
      - ./consul/data2:/consul/data
    command: "agent -server -ui -join consul-server1 -disable-host-node-id -client 0.0.0.0"
    depends_on:
      - consul-server1
  consul-server3:
    image: consul:latest
    hostname: "consul-server3"
    ports:
      - "8502:8500"
      - "53"
    volumes:
      - ./consul/data3:/consul/data
    command: "agent -server -ui -join consul-server1 -disable-host-node-id -client 0.0.0.0"
    depends_on:
      - consul-server1
  consul-node1:
    image: consul:latest
    hostname: "consul-node1"
    command: "agent -join consul-server1 -disable-host-node-id"
    depends_on:
      - consul-server1
  consul-node2:
    image: consul:latest
    hostname: "consul-node2"
    command: "agent -join consul-server1 -disable-host-node-id"
    depends_on:
      - consul-server1
```