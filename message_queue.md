# MESSAGE_QUEUE
## JMS规范
### Connection
开启会话
### Session
创建destination，messageProducer, message, messageCustomer, queueBrowser
### Destination
空接口，有Queue，Topic的实现
在queue模式下，每个消息只能被一个消息接收者接收
在topic模式下，每个消息所有订阅者都能接收
### Message
发送和接收的消息对象
### MessageProducer
消息发送者，发送message，能设置分发模式，优先级，消息存活时间
### MessageConsumer
消息接受者，接收message，能设置消息 MessageListener，在收到消息时响应

## ActiveMQ
遵从JMS规范
### ActiveMQConnectionFactory
创建连接


## Kafka
### Linux

有两种协议 PLAINTEXT 和 SSL
没有配置 advertised.listeners 时将使用 listeners 配置
advertised.listeners 向 zookeeper 注册 broker 的连接信息，不参与创建连接
listeners broker 创建启动 socket 的配置
需要配置 advertised.listeners java 代码才能得到响应

### java
#### KafkaProducer
推送ProducerRecord，在ProducerRecord中申明topic
#### KafkaConsumer
接收ConsumerRecord，需预先预定主题（调用 subscribe(topics) 方法)


