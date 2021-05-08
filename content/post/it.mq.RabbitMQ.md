

+++

title = "RabbitMQ"
description = "RabbitMQ"
tags = ["it", "mq"]

+++



# RabbitMQ

https://www.rabbitmq.com/

# client

## go

```shell
go get -u github.com/streadway/amqp
```

```go
package rabbitmq

const URL = "amqp://guest:guest@127.0.0.1:5672/" //格式：amqp://账号:密码@地址:端口/虚拟主机

type RabbitMQ struct {
	url string //连接信息

	conn    *amqp.Connection //连接
	channel *amqp.Channel    //通道

	queueName    string //queue名
	exchangeName string //exchange名
	routingKey   string //routing key
}

/*RabbitMQ实例化*/
func NewRabbitMQ(queueName string, exchangeName string, routingKey string) *RabbitMQ {
	rabbitMQ := &RabbitMQ{url: URL, queueName: queueName, exchangeName: exchangeName, routingKey: routingKey}
	var err error

	rabbitMQ.conn, err = amqp.Dial(rabbitMQ.url) //获取connection
	rabbitMQ.failOnError(err, "创建连接错误！")

	rabbitMQ.channel, err = rabbitMQ.conn.Channel() //获取channel
	rabbitMQ.failOnError(err, "获取channel失败")
	return rabbitMQ
}

/*断开通道和连接*/
func (r *RabbitMQ) Destroy() {
	r.channel.Close()
	r.conn.Close()
}

/*错误处理
包括可以将错误写入日志、数据库、发送邮件等。
不用于代码错误处理*/
func (r *RabbitMQ) failOnError(err error, message string) {
	if err != nil {
		log.Fatalf("%s:%s", message, err)
		panic(fmt.Sprintf("%s:%s", message, err))
	}
}

/*消息处理*/
func handleMessages(messages <-chan amqp.Delivery) {
	forever := make(chan bool)
	go func() {
		for msg := range messages {
			//消息处理
			log.Printf("[*] Receieved a message:%s for messages, To exit press CTRL+C", msg.Body)
		}
	}()
	log.Printf("[*] Waiting for messages, To exit press CTRL+C")
	<-forever
}
```



### 工作模式

#### simple

简单模式：生产者→消息队列→消费者。一个生产者，一个消费者。消息被消费一次就删除

```go
package rabbitmq

/*---------------------------Simple---------------------------*/

/*Simple模式RabbitMQ实例化*/
func NewRabbitMQSimple(queueName string) *RabbitMQ {
	return NewRabbitMQ(queueName, "", "") //""表示不指定，将默认使用RabbitMQ初始自带的name=(AMQP default)的Exchange
}

/*Simple模式生产*/
func (r *RabbitMQ) PublishSimple(message string) {
	//1.申请队列。如果队列存在则获取，如果队列不存在则自动创建并获取。保证队列存在，消息可以发送到队列
	_, err := r.channel.QueueDeclare(
		r.queueName, //name：队列名
		false,       //durable：消息是否持久化
		false,       //autoDelete：是否自动删除。当最后一个消费者断开连接后，是否自动删除队列中的消息
		false,       //exclusive：是否具有排他性。true时，该队列无法被其它用户访问
		false,       //noWait：是否不等待。发送消息时是否不等待RabbitMQ服务器响应，false则需要等待
		nil,         //args：额外属性
	)
	if err != nil {
		fmt.Println(err)
	}

	//2.消息发布到队列
	r.channel.Publish(
		r.exchangeName, //exchange：exchange名
		r.queueName,    //queue：queue名
		false,          //mandatory：是否托管。true时，会根据Exchange的类型和routeKey规则，寻找符合条件的队列，找不到，会将发布的消息返还给消息生产者
		false,          //immediate：是否直属。true时，当exchange发送消息到队列后发现队列上没有绑定消费者，则将把发送的消息返还给消息生产者

		amqp.Publishing{ //消息发布设置
			ContentType: "text/plain",    //消息内容类型
			Body:        []byte(message), //消息体
		},
	)
}

/*Simple模式消费*/
func (r *RabbitMQ) ConsumeSimple() {
	//1.申请队列。如果队列存在则获取，如果队列不存在则自动创建并获取。保证队列存在，消息可以发送到队列
	_, err := r.channel.QueueDeclare(r.queueName, false, false, false, false, nil)
	if err != nil {
		fmt.Println(err)
	}

	//2.消息接收
	messages, err := r.channel.Consume(
		r.queueName, //queue：队列名。如果不指定将随机生成队列名
		"",          //consumer：消费者。用来区分多个消费者。不指定
		true,        //autoAck：是否自动命令正确应答，默认true。true时，消费消费消息后自动告诉RabbitMQ服务器消息正确消费完毕，mq服务器就可以去做删除消息等操作；false时，需要手动实现回调函数
		false,       //exclusive
		false,       //noLocal：是否非本地。true时，不能将同一个连接中发布的消息发送给该连接的消费者
		false,       //noWait
		nil,         //args
	)
	if err != nil {
		fmt.Println(err)
	}

	//3.消息处理
	handleMessages(messages)
}
```

测试：先启动消费者，再启动生产者

```go
/*生产者*/
func main() {
   rabbitMQ := RabbitMQ.NewRabbitMQSimple("yuanyaQueueSimple")
   rabbitMQ.PublishSimple("Hello yuanya！")
   fmt.Println("发送成功！")
}
```

```go
/*消费者*/
func main() {
   rabbitMQ := RabbitMQ.NewRabbitMQSimple("yuanyaQueueSimple")
   rabbitMQ.ConsumeSimple()
}
```



#### work

工作模式：生产者→消息队列→消费者1、消费者2...消费者N。一个生产者，多个消费者。一个消息只能被一个消费者消费一次。默认是轮询模式的负载均衡

使用场景：生产消息的速度大于消费消息的速度，来增加系统的性能和处理能力，起到负载均衡

代码实现与Simple模式一致，只是创建多创建几个消费者即可



#### publish/subscribe

发布订阅模式：生产者→exchange→queue1、queue2...queueN→consumer1、consumer2...consumerN。

1. 生产者：生产者发送消息到exchange
2. exchange："fanout"，发散类型，即广播。exchange广播消息给所有该queue
3. queue：queue发送给对应消费者。默认一个queue对应一个consumer

```go
package rabbitmq

/*---------------------------Publish/Subscribe---------------------------*/

/*Publish/Subscribe模式RabbitMQ实例化*/
func NewRabbitMQPubSub(exchangeName string) *RabbitMQ {
	return NewRabbitMQ("", exchangeName, "")
}

/*Publish/Subscribe模式生产*/
func (r *RabbitMQ) PublishPubSub(message string) {
	//1.Exchange声明。将尝试创建exchange
	err := r.channel.ExchangeDeclare(
		r.exchangeName, //name：交换名
		"fanout",       //kind：交换类型
		true,           //durable
		false,          //autoDelete
		false,          //internal：内部的。
		false,          //noWait
		nil,            //args
	)
	if err != nil {
		fmt.Println(err)
	}

	//2.消息发布到队列
	r.channel.Publish(r.exchangeName, "", false, false,
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		},
	)
}

/*Publish/Subscribe模式消费*/
func (r *RabbitMQ) ConsumePubSub() {
	//1.尝试创建exchange
	err := r.channel.ExchangeDeclare(r.exchangeName, "fanout", true, false, false, false, nil)
	if err != nil {
		fmt.Println(err)
	}

	//2.申请队列
	queue, err := r.channel.QueueDeclare("", false, false, true, false, nil)
	if err != nil {
		fmt.Println(err)
	}

	//3.绑定Queue、Exchange
	r.channel.QueueBind(queue.Name, "", r.exchangeName, false, nil)

	//4.消息接收
	messages, err := r.channel.Consume(queue.Name, "", true, false, false, false, nil)
	if err != nil {
		fmt.Println(err)
	}

	//5.消息处理
	handleMessages(messages)
}
```

测试：先启动消费者，再启动生产者

```go
/*生产者*/
func main() {
   rabbitMQ := RabbitMQ.NewRabbitMQSimple("yuanyaQueuePubSub")
   rabbitMQ.PublishSimple("Hello yuanya！")
   fmt.Println("发送成功！")
}
```

```go
/*消费者1。可以接收消息*/
func main() {
   rabbitMQ := RabbitMQ.NewRabbitMQSimple("yuanyaQueuePubSub")
   rabbitMQ.ConsumeSimple()
}
```

```go
/*消费者2。可以接收消息*/
func main() {
   rabbitMQ := RabbitMQ.NewRabbitMQSimple("yuanyaQueuePubSub")
   rabbitMQ.ConsumeSimple()
}
```



#### routing

路由模式：一个消息可以被多个消费者获取，并且消息的目标队列可以被生成者指定

1. producer：producer发送消息到exchange
2. exchange："direct"，指导类型。发送消息到指定队列
3. queue：queue发送给对应消费者。默认一个queue对应一个consumer

```go
package rabbitmq

/*---------------------------Routing---------------------------*/

/*Routing模式RabbitMQ实例化*/
func NewRabbitMQRouting(exchangeName, routingKey string) *RabbitMQ {
   return NewRabbitMQ("", exchangeName, routingKey)
}

/*Routing模式生产*/
func (r *RabbitMQ) PublishRouting(message string) {
   //1.Exchange声明
   err := r.channel.ExchangeDeclare(r.exchangeName, "direct", true, false, false, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //2.消息发布到队列
   r.channel.Publish(r.exchangeName, r.routingKey, false, false,
      amqp.Publishing{
         ContentType: "text/plain",
         Body:        []byte(message),
      },
   )
}

/*Routing模式消费*/
func (r *RabbitMQ) ConsumeRouting() {
   //1.Exchange声明
   err := r.channel.ExchangeDeclare(r.exchangeName, "direct", true, false, false, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //2.申请队列
   queue, err := r.channel.QueueDeclare("", false, false, true, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //3.绑定Queue、Exchange
   r.channel.QueueBind(queue.Name, r.routingKey, r.exchangeName, false, nil, )

   //4.消息接收
   messages, err := r.channel.Consume(queue.Name, "", true, false, false, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //5.消息处理
   handleMessages(messages)
}
```

测试：先启动消费者，再启动生产者

```go
/*生产者*/
func main() {
   rabbitMQ1 := RabbitMQ.NewRabbitMQRouting("yuanyaExchangeRouting", "yuanyaRoutingKey1")
   rabbitMQ2 := RabbitMQ.NewRabbitMQRouting("yuanyaExchangeRouting", "yuanyaRoutingKey2")
   rabbitMQ1.PublishRouting("Hello yuanya！1")
   rabbitMQ2.PublishRouting("Hello yuanya！2")
   fmt.Println("发送成功！")
}
```

```go
/*消费者1。可以接收yuanyaRoutingKey1对应的queue的消息*/
func main() {
	rabbitMQ := RabbitMQ.NewRabbitMQRouting("yuanyaExchangeRouting", "yuanyaRoutingKey1")
	rabbitMQ.ConsumeRouting()
}
```

```go
/*消费者2。yuanyaRoutingKey1和2对应的queue的消息都可以接收*/
func main() {
	rabbitMQ1 := RabbitMQ.NewRabbitMQRouting("yuanyaExchangeRouting", "yuanyaKey2")
	rabbitMQ2 := RabbitMQ.NewRabbitMQRouting("yuanyaExchangeRouting", "yuanyaKey1")
	go func() {rabbitMQ1.ConsumeRouting()}()
	rabbitMQ2.ConsumeRouting()
}
```



#### topic

主题模式：一个消息被多个消费者获取，消息的目标Queue可用RoutingKey以通配符的方式指定，如\*.orange.\*、lazy.#，#表示一个或多个词，*表示一个词，Exchange的Type必须是"topic"。比如消费者指定routing key为#，则接收所有消息，生产者随便定设置routing key如a.b.c都能被消费者接收。实现时除了把生产者、消费者获取Exchange时type指定为topic，其它代码一摸一样

```go
package rabbitmq

/*---------------------------Topic---------------------------*/

/*Topic模式RabbitMQ实例化*/
func NewRabbitMQTopic(exchangeName, routingKey string) *RabbitMQ {
   return NewRabbitMQ("", exchangeName, routingKey)
}

/*Topic模式生产*/
func (r *RabbitMQ) PublishTopic(message string) {
   //1.Exchange声明
   err := r.channel.ExchangeDeclare(r.exchangeName, "topic", true, false, false, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //2.消息发布到队列
   r.channel.Publish(r.exchangeName, r.routingKey, false, false,
      amqp.Publishing{
         ContentType: "text/plain",
         Body:        []byte(message),
      },
   )
}

/*Topic模式消费*/
func (r *RabbitMQ) ConsumeTopic() {
   //1.Exchange声明
   err := r.channel.ExchangeDeclare(r.exchangeName, "topic", true, false, false, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //2.申请队列
   queue, err := r.channel.QueueDeclare("", false, false, true, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //3.绑定Queue、Exchange
   r.channel.QueueBind(queue.Name, r.routingKey, r.exchangeName, false, nil, )

   //4.消息接收
   messages, err := r.channel.Consume(queue.Name, "", true, false, false, false, nil)
   if err != nil {
      fmt.Println(err)
   }

   //5.消息处理
   handleMessages(messages)
}
```

测试：先启动消费者，再启动生产者

```go
/*生产者*/
func main() {
   rabbitMQ := RabbitMQ.NewRabbitMQSimple("yuanyaQueueSimple")
   rabbitMQ.PublishSimple("Hello yuanya！")
   fmt.Println("发送成功！")
}
```

```go
/*消费者*/
func main() {
   rabbitMQ := RabbitMQ.NewRabbitMQSimple("yuanyaQueueSimple")
   rabbitMQ.ConsumeSimple()
}
```

# hello

## 安装

### linux

安装Erlang，因为RabbitMQ依赖Erlang

```shell
wget https://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm #下载
rpm -ivh erlang-19.0.4-1.el7.centos.x86_64.rpm #安装
```

安装RabbitMQ

```shell
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-3.6.15-1.el7.noarch.rpm #下载
rpm -ivh rabbitmq-server-3.6.15-1.el7.noarch.rpm #安装
systemctl start rabbitmq-server #启动
rabbitmqctl stop #停止
```



### docker

带有management的是带有web控制台的

```shell
docker pull rabbitmq:3.7.24-management 
docker run -d -p 5672:5672 -p 15672:15672 rabbitmq:3.7.24-management
```



## API

##### 插件

插件列表：显示的插件列表中带有E*或e\*的表示已启用的插件，大写表示主插件，如[E\*] rabbitmq_management

```shell
rabbitmq-plugins list #rabbitmq插件列表
```

启用插件

```shell
rabbitmq-plugins enable rabbitmq_management
```

停用插件

```shell
rabbitmq-plugins disable rabbitmq_management
```



## 控制台

http://127.0.0.1:15672，缺省账号密码guest

- Overview：概览
- Connections：连接。用于连接rabbitmq的客户端和服务器。rabbitmq-client通过格式为"amqp://账号:密码@地址:端口/虚拟主机" 的url来获取服务器的Connection
- Channels：通道。用于客户端与服务器通信。客户端通过一个有效Connection获取Channel
- Exchanges：交换。等于根据规则转发信息的交换机。Exchange根据其Type对应规则转发给正确的Queue。未指定Exchange时，则将默认使用RabbitMQ初始自带的Name="(AMQP default)"的Exchange转发给指定的Queue
  - Name：Exchange名字。
    - "(AMQP default)"：RabbitMQ初始自带的其中一个Exchange，Type为"direct"。该Exchange不能绑定Queue，但是可以通过QueueName转发消息给任何Queue
  - Type：Exchange类型
    - "direct"：指导。转发给该Exchange下的指定RoutingKey对应的所有Queue。"direct"类型下，必须指定RoutingKey
    - "fanout"：发散。转发给该Exchange下的所有Queue，即广播。无需指定Queue
    - "topic"：主题。
    - "headers"
  - Bindings：绑定。绑定Queue与Exchange
    - To：QueueName，队列名。即当前Exchange绑定的Queue
    - Routing key：路由键。等于对当前Exchange绑定的Queue进行分组的分组标识。一个Queue只能对应一个RoutingKey。RoutingKey只作用于Exchange内，不影响其它Exchange的RoutingKey
    - Arguments：参数
- Queues：队列。如果不指定Queue。未指定Queue时，将根据Exchange的配置来决定是否报错、是否发散转发、是否指导转发等
  - Name：Queue名字
  - Bindings：同Exchange的Bindings
- Admin：
  - Users：用户
  - Virtual Hosts：虚拟主机。可以有效隔离开发、测试等各种环境，
  - Feature Flags
  - Policies
  - Limits
  - Cluster

注意：传入空字符串""表示不指定

## 核心

- Connection：连接
- Channel：通道。用来通信。一个连接可对应多个通道
- Exchange：交换。等于根据规则转发信息的交换机。producer发布消息，消息首先发送到Exchange，
  - 
- Queue：队列

- 流程
  - producer发布消息，可以指定queueName、exchangeName、routingKey
    1. 消息首先根据exchangeName发送到指定的Exchange。如果未指定Exchange（exchangeName=""），则将使用默认的Name为(AMQP default)、Type为direct的Exchange
    2. Exchange根据其Type、绑定的routingKey，将消息转发给正确的Queue。

