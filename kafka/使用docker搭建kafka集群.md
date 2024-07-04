​	在使用docker搭建kafka集群之前, 先要了解在不使用docker时如何搭建kafka集群.

	1. 一台zookeeper
	1. 三台kafka



​	zookeeper不需要修改配置文件.

​	kafka需要修改配置文件server.properties:

1. broker.id=1
2. listeners=PLAINTEXT://:9092 
3. zookeeper.connect=localhost:2181



​	因此, 在使用docker搭建kafka集群时, 也需要考虑到这些因素.

​	下面编写docker compose文件, 搭建kafka集群:

```yaml
version: "2"

services:
  zookeeper:
    name: zookeeper
    image: docker.io/bitnami/zookeeper:3.9
    ports:
      - "2181:2181"
    volumes:
      - "zookeeper_data:/bitnami"
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka-1:
  	name: kafka-1
    image: docker.io/bitnami/kafka:3.4
    ports:
      - "9092:9092"
    volumes:
      - "kafka_data_1:/bitnami"
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
    net-works:
    	- kafka_group
    depends_on:
      - zookeeper
	kafka-2:
  	name: kafka-2
    image: docker.io/bitnami/kafka:3.4
    ports:
      - "9093:9092"
    volumes:
      - "kafka_data_2:/bitnami"
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
    net-works:
    	- kafka_group
    depends_on:
      - zookeeper
	kafka-3:
  	name: kafka-3
    image: docker.io/bitnami/kafka:3.4
    ports:
      - "9094:9092"
    volumes:
      - "kafka_data_4:/bitnami"
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
    net-works:
    	- kafka_group
    depends_on:
      - zookeeper

volumes:
  zookeeper_data:
    driver: local
  kafka_data:
    driver: local
    
networks:
	kafka_group:
```

