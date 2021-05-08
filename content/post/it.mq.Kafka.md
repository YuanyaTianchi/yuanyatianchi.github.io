

+++

title = "Kafka"
description = "Kafka"
tags = ["it", "mq"]

+++



# Kafka

http://kafka.apache.org/



# client

## go

链接kafka使用第三方库 https://github.com/Shopify/sarama ，`sarama v1.20`之后的版本加入了`zstd`压缩算法，编译需要用到cgo，win上需要下载 http://mingw-w64.org/doku.php/download ，当然也可以使用`v1.20`之前的版本

```shell
got get -u github.com/Shopify/sarama
```

```GO
/*kafka地址列表*/
var KAFKA_ADDR_LIST = []string{"127.0.0.1:9092"}
```

```go
/*生产者*/
func Producer() {
	var (
		config          *sarama.Config          //客户端配置
		syncProducer    sarama.SyncProducer     //Producer客户端
		producerMessage *sarama.ProducerMessage //producer消息
	)

	//构造配置
	config = sarama.NewConfig()
	config.Producer.RequiredAcks = sarama.WaitForAll          //发送完数据需要leader和follow都确认
	config.Producer.Partitioner = sarama.NewRandomPartitioner //新选出一个partition
	config.Producer.Return.Successes = true                   //成功交付的消息将在success channel返回

	//通过配置构造producer客户端
	syncProducer, err := sarama.NewSyncProducer(KAFKA_ADDR_LIST, config)
	if err != nil {
		fmt.Println("producer closed, err:", err)
		return
	}
	defer syncProducer.Close()

	//构造消息
	producerMessage = &sarama.ProducerMessage{
		Topic: "web_log",                                  //指定Topic
		Value: sarama.StringEncoder("this is a test log"), //消息内容
	}

	//发送消息
	pid, offset, err := syncProducer.SendMessage(producerMessage)
	if err != nil {
		fmt.Println("send msg failed, err:", err)
		return
	}
	fmt.Printf("pid:%v offset:%v\n", pid, offset)
}
```

```go
/*消费者*/
func Consumer() (err error) {
	var (
		consumer sarama.Consumer //consumer客户端
	)

	//构造consumer实例
	consumer, err = sarama.NewConsumer(KAFKA_ADDR_LIST, nil)
	if err != nil {
		fmt.Printf("fail to start consumer, err:%v\n", err)
		return err
	}

	//通过topic取到所有的分区
	partitionList, err := consumer.Partitions("web_log")
	if err != nil {
		fmt.Printf("fail to get list of partition:err%v\n", err)
		return
	}
	fmt.Println(partitionList)

	//遍历所有的分区
	for partition := range partitionList {
		//针对每个分区创建一个对应的分区消费者
		partitionConsumer, err := consumer.ConsumePartition("web_log", int32(partition), sarama.OffsetNewest)
		if err != nil {
			fmt.Printf("failed to start consumer for partition %d,err:%v\n", partition, err)
			return
		}
		defer partitionConsumer.AsyncClose()
		
		// 异步从每个分区消费信息
		go func(sarama.PartitionConsumer) {
			for msg := range partitionConsumer.Messages() {
				fmt.Printf("Partition:%d Offset:%d Key:%v Value:%v", msg.Partition, msg.Offset, msg.Key, msg.Value)
			}
		}(partitionConsumer)
	}
	
	return nil
}
```



# hello

```bash
#通过docker-compose启动
docker-compose up -d #默认执行./docker-compose.yml
docker-compose -f ./docker-compose-kafka.yml up -d #指定yml文件
```

```Yaml
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper01
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/zookeeper
    container_name: kafka01
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 172.16.69.166
      KAFKA_ZOOKEEPER_CONNECT: zookeeper01:2181 #这里的zookeeper由上面container_name指定
    depends_on:
      - zookeeper01
```



3zookeeper+3kafka的docker-compose.yml

```yaml
version: '3.4' 

services: 
  zoo1: 
    image: zookeeper:3.4 
    restart: always 
    hostname: zoo1 
    container_name: zoo1 
    ports: 
      - 2184:2181 
    volumes: 
      - "/home/zk/workspace/volumes/zkcluster/zoo1/data:/data" 
      - "/home/zk/workspace/volumes/zkcluster/zoo1/datalog:/datalog" 
    environment: 
      ZOO_MY_ID: 1 
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888 
      networks: 
        kafka: 
          ipv4_address: 172.30.0.11 
  zoo2: 
    image: zookeeper:3.4 
    restart: always 
    hostname: zoo2 
    container_name: zoo2 
    ports: 
      - 2185:2181 
    volumes: 
      - "/home/zk/workspace/volumes/zkcluster/zoo2/data:/data" 
      - "/home/zk/workspace/volumes/zkcluster/zoo2/datalog:/datalog" 
    environment: 
      ZOO_MY_ID: 2 
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888 
    networks: 
      kafka: 
        ipv4_address: 172.30.0.12 
  zoo3: 
    image: zookeeper:3.4 
    restart: always 
    hostname: zoo3 
    container_name: zoo3 
    ports: 
      - 2186:2181 
    volumes: 
      - "/home/zk/workspace/volumes/zkcluster/zoo3/data:/data" 
      - "/home/zk/workspace/volumes/zkcluster/zoo3/datalog:/datalog" 
    environment: 
      ZOO_MY_ID: 3 
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888 
    networks: 
      kafka: 
        ipv4_address: 172.30.0.13 

  kafka1: 
    image: wurstmeister/kafka 
    restart: always 
    hostname: kafka1 
    container_name: kafka1 
    privileged: true 
    ports: 
      - 9092:9092 
    environment: 
      KAFKA_ADVERTISED_HOST_NAME: kafka1 
      KAFKA_LISTENERS: PLAINTEXT://kafka1:9092 
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092 
      KAFKA_ADVERTISED_PORT: 9092 
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181 
    volumes: 
      - /home/zk/workspace/volumes/kafkaCluster/kafka1/logs:/kafka 
    networks: 
      kafka: 
        ipv4_address: 172.30.1.11 
    extra_hosts: 
      - "zoo1:172.30.0.11" 
      - "zoo2:172.30.0.12" 
      - "zoo3:172.30.0.13" 
    depends_on: 
      - zoo1 
      - zoo2 
      - zoo3 
    external_links: 
      - zoo1 
      - zoo2 
      - zoo3 
  kafka2: 
    image: wurstmeister/kafka 
    restart: always 
    hostname: kafka2 
    container_name: kafka2 
    privileged: true 
    ports: 
      - 9093:9093 
    environment: 
      KAFKA_ADVERTISED_HOST_NAME: kafka2 
      KAFKA_LISTENERS: PLAINTEXT://kafka2:9093 
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9093 
      KAFKA_ADVERTISED_PORT: 9093 
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181 
    volumes: 
      - /home/zk/workspace/volumes/kafkaCluster/kafka2/logs:/kafka 
    networks: 
      kafka: 
        ipv4_address: 172.30.1.12 
    extra_hosts: 
      - "zoo1:172.30.0.11" 
      - "zoo2:172.30.0.12" 
      - "zoo3:172.30.0.13" 
    depends_on: 
      - zoo1 
      - zoo2 
      - zoo3 
    external_links: 
      - zoo1 
      - zoo2 
      - zoo3 
  kafka3: 
    image: wurstmeister/kafka 
    restart: always 
    hostname: kafka3 
    container_name: kafka3 
    privileged: true 
    ports: 
      - 9094:9094 
    environment: 
      KAFKA_ADVERTISED_HOST_NAME: kafka3 
      KAFKA_LISTENERS: PLAINTEXT://kafka3:9094 
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9094 
      KAFKA_ADVERTISED_PORT: 9094 
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181 
    volumes: 
      - /home/zk/workspace/volumes/kafkaCluster/kafka3/logs:/kafka 
    networks: 
      kafka: 
        ipv4_address: 172.30.1.13 
    extra_hosts: 
      - "zoo1:172.30.0.11" 
      - "zoo2:172.30.0.12" 
      - "zoo3:172.30.0.13" 
    depends_on: 
      - zoo1 
      - zoo2 
      - zoo3 
    external_links: 
      - zoo1 
      - zoo2 
      - zoo3 

networks: 
  kafka: 
    external: 
      name: kafka
```





Kafka是一个分布式的基于发布/订阅模式的消息队列(MessageQueue)，主要应用于大数据实时处理领域。

- 消息队列
  - 
    - 解耦：允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。
    - 可恢复性：系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所
      以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
    - 缓冲。有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。
    - 灵活性&峰值处理能力。
      在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。
      如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列
      能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。
    - 异步通信。
      很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户
      把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要
      的时候再去处理它们。。
  - 模式
    - 点对点模式(一对一，消费者主动拉取数据，消息收到后消息清除) 
      消息生产者生产消息发送到Queue中,然后消息消费者从Queue中取出并且消费消息。心
      消息被消费以后，queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息。.
      Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。
    - (2)发布/订阅模式(一对多，消费者消费数据之后不会清除消息) +
      消息生产者(发布)将消息发布到topic中，同时有多个消息消费者(订阅)消费该消
      息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。
      - mq主动推出。如公众号。在大数据量下容易使得消费者方崩溃或资源浪费，如生产者消息生产速度是50m/s，消费者A接收消息速度是100m/s，消费者B是10m/s，则B崩溃，A浪费
      - 消费者主动拉取。缺点是消费者需要维护一个长轮询来查看mq中是否有新消息可以拉取。kafka就是如此。

- 架构：图1
  - topic主题，有分区partition，每个分区（一个leader）都有备份（若干follower）。kafka集群中follower只是leader的备份，不能在同一台机器上，消息生产者和消息消费者只找leader，而不是像其它一些集群随机从leader和follower中择其一访问。
  - 消费者可以分组，一组中的消费者是竞争关系，一个分区的一条消息只能被一个组中的一个消费者消费一次，如果一个topic有两个分区，但是订阅它的消费者只有一个，则该topic发布的一条消息到两个分区中，这一个消费者可以从两个分区中各消费一次该消息，即该消费者可以消费两次同一条消息，提高了消费能力。一个topic的分区与订阅该topic的消费者是m对n的关系，那么最好的情况是m=n
  - 消费者消费到一半挂掉，重启需要接着上一次消费mq中的消息，那么消费者就需要记录上一次消费到mq的什么位置offset，内存中当然有一份记录以便正常运转使用，但是挂掉内存就释放了，而kafka0.9版本之前的外部存储就是保存在zookeeper中的，0.9及版本之后保存在kafka本地某个topic里面（不是本地磁盘，kafka的消息是存在本地磁盘的，缺省保留168小时即7天）。因为消费者本来就要维护与kafka集群的连接，如果offset存储在zookeeper中则还需要维护与zookeeper的连接，并且交互频率与消息拉取频率一致，太过于频繁





