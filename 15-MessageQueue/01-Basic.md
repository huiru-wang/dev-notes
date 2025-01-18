
https://www.geeksforgeeks.org/message-queues-system-design/
# What is MessageQueue

A message queue is a form of Asynchronous communication and data transfer mechanism. For messages sent between various systems inside a broader software architecture, it serves as a temporary storage and routing system.

> 消息队列是一种异步通信和数据传输机制
> 在广泛的软件架构中的系统间发送消息，充当临时存储和路由系统

## Key Components

![](/images/MessageQueue-mq.png)

1. Producer: any program that produces data for sharing and send to the message queue can be considered producers.
2. MessageQueue: the messages stored and managed by a data structure util the consumers consume them. It serves as buffer between producers and consumers.
3. Consumer: consume and processed the messages from the message queue.
4. Message: a message can be defined as a unit of data that is transferred between producers and consumers.

> 关键的组件就是：生产者、队列、消费者、消息；
# Benefits

In a distributed system, message queue is one of the most important component.
## 1. Asynchronous Communication
1. Asynchronous Communication: In scenarios where the core workflow does not need to wait synchronously for non-core operations, the non-core operations can be made asynchronous which can improves the response speed of the core workflow.
## 2. Data Peak Shaving

When a system is unable to handle massive requests or when it is not necessary to process all incoming traffic immediately, message queue can act as a buffer.

> 当一个系统不能立即处理大量的请求或不需要立即处理所有到来的流量，此时MQ可以被当作一个缓冲区

For example, if the system can handle 2000 requests per second, but during the peek time of 5000 requests per second, the system can leave 3000 requests to be stored in the MQ, preventing the system from breaking down during peek time. The requests in the MQ can be consumed by the server at a constant rate.

> 一个系统能够每秒处理2000请求，但是在流量高峰每秒有5000个请求，系统就可以将3000请求存储在MQ中，以防止宕机，在MQ中的请求可以被服务器以恒定的速率消费

## 3. Data Reuse

When a piece of data needs to be consumed by multiple services, the message queue can distribute the message to different services. At the same time, that ensures data consistency across different parts of the system, because all modules receive the same data from the message queue.

> 当一份数据需要被多个服务消费时，消息队列可以将消息分发到不同的服务，同时也确保了数据一致性，因为所有的系统内模块都从MQ中消费相同的数据

## 4. Decoupling

Message queues decouple applications from each other, allowing them to be developed independently. This makes systems more flexible, scalable and easier to maintain.

>MQ可以解耦系统，允许它们独立开发，这样做让系统更弹性、更具扩展性、更好维护

## 5. Parallel Processing

A large task can be split into many smaller tasks, and each task can be sent as a message to the MQ, so that the tasks can be processed in parallel.

> 一个大任务可以被分割成多个小任务，每个任务都可以当成消息投递到MQ，这样可以让这些任务并发处理
## 6. Reliability

- Message-Persistence
Message queue middleware typically offer message-persistence features. When a message is sent to the MQ, it is stored in a reliable storage. This ensures that even if the MQ server restarts, the messages are not lost.

> 消息中间件通常提供消息持久化机制，当一个消息被发送到队列中，它就会被存储到一个可靠存储中，这保证了即使消息队列服务重启，消息仍然不会丢失

- Delivery Guarantees
Message queue middleware provide different levels of delivery guarantees.

>MQ中间件通常提供不同等级的投递保证

- Redundancy and Fault-Tolerance
Multiple MQ servers can be set up in a cluster, and messages can be replicated across these servers.
They can detect failures of producers, consumers. When a consumer fails, the MQ can stop sending messages to that particular consumer and route them to other available consumers.

> 大部分MQ服务可以以集群方式启动，消息可以备份在不同的Server节点中
> 它们可以检测生产者、消费者的故障。当消费者发生故障时，MQ 可以停止向该特定消费者发送消息并将其路由到其他可用的消费者。

# Potential Problems

We need to pay attention to some potentail problems when adding MQ middleware to a distributed system:
## 1. Consumption Idempotency
幂等问题

幂等问题：重复生产/重复消费
生产端幂等：
1. 使用唯一ID保证消息幂等；
2. 业务自己保证发送幂等，每次发送消息都检查消息是否已经发送/或使用事务保证；

消费端：
1. 结合DB的幂等性；使用覆盖代替计算；
2. 业务上进行判重等操作来保证；
## 2. Message Loss

要保证消息可靠传输，要从消息链路每个地方保证：
1. 生产者发送消息到消息队列时丢失；
2. 消息队列未能持久化消息，宕机丢失；
3. 消息消费时，未能消费成功，MQ清除消息或者宕机丢失；

三个方面去考虑：
1. 保证生产端可靠投递（<font color="#de7802">同步投递</font>）
	- 采用同步方式发送消息，确保收到MQ的成功响应，保证投递成功；
2. 保证MQ端的消息副本同步、持久化；（<font color="#de7802">TP模式</font>）
	- MQ在收到消息后，保证消息同步到各个MQ副本上，再响应生产端；副本同步成功已经可以保证消息不丢失了；
	- 如果要完全保证，还需要持久化消息，但是会拉低性能；
3. 消费端保证消费完成，从MQ中Commit指定消息（<font color="#de7802">同步Commit</font>）
	- 当消费端，消费完成，应当响应MQ，去进行对应消息的消费Commit；


## 3. Message Block

没办法，要消费，就只能提高消费端的消费能力；
1. 增加消费者线程、进程；
2. 优化业务消费逻辑；

## 4. Message out-of Order

1. <font color="#de7802">单线程消费</font>：将所有消息都由同一个线程来消费，这样就可以保证消息的顺序性。但是这样会影响系统的吞吐量和并发性能。
2. <font color="#de7802">按分区消费</font>：将不同分区的消息分配给不同的消费者线程来处理，每个消费者只处理自己分区的消息。这样可以保证每个分区内的消息顺序消费；
   但是不同分区之间的消息仍然有可能出现乱序。
4. 结合业务特点，保证处理消息的顺序性；比如先入库，再顺序消费数据库消息；
# MQ Architecture

## Point-to-point

Simplest type of Message queue.

## Publish-Subscribe

TODO

## Routing Strategy

1. Topic-Based Routing
2. Direct Routing
3. content-Based Routing

TODO


# Middleware

1. kafka
2. Pulsar
3. RocketMQ（MetaQ）
4. RabbitMQ
5. Redis

