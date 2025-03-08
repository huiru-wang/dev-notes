---
title: 分布式系统中消息队列的工作方式和特性
category: MessageQueue
tags:
  - MessageQueue
publishedAt: 2022-11-12
description: 分布式消息队列的关键组件
---

https://www.geeksforgeeks.org/message-queues-system-design/
# 消息队列

A message queue is a form of Asynchronous communication and data transfer mechanism. For messages sent between various systems inside a broader software architecture, it serves as a temporary storage and routing system.

> 消息队列是一种异步通信和数据传输机制
> 在广泛的软件架构中的系统间发送消息，充当临时存储和路由系统

## 关键组件

![](/images/MessageQueue-mq.png)

1. 生产者Producer: any program that produces data for sharing and send to the message queue can be considered producers.
2. 消息队列MessageQueue: the messages stored and managed by a data structure util the consumers consume them. It serves as buffer between producers and consumers.
3. 消费者Consumer: consume and processed the messages from the message queue.
4. 消息Message: a message can be defined as a unit of data that is transferred between producers and consumers.

# 消息投递语意

1. At Most Once
2. At Least Once
3. Exactly Only Once

在分布式系统中，消息的重复是可以容忍的，甚至某些情况下是需要的（消费者消费过程中宕机，未Commit），但是消息的丟失往往是不能容忍的。因此"At Most Once"基本不能满足要求。

因此只需要讨论下"At Least Once"和"Exactly Only Once"：

（1）实现复杂度考虑：
- At Least Once：只需要通过重试机制保证消息不丟失；
	- 生产者发送消息，如果broker未确认、超时，则重试；
	- Broker持久化消息后，就保证了最少一次；消费失败也可重新投递；
- Exactly Only Once：需要全链路（生产、传输、消费）严格协调，避免重复：
	- 生产者消息去重（消息唯一id + 事务日志）
	- Broker全局去重（需要记录所有消息的状态）
	- 消费者幂等处理（通过全局唯一标识）

（2）性能考虑：
- At Least Once：仅依赖本地重试、确认机制，对吞吐量影响很小，能够有更高的并发能力；
- Exactly Only Once：全局锁、事务日志、协调协议等等都会大幅降低吞吐量；（Kafka开启事务后吞吐量下降20%~30%）

（3）分布式一致性考虑：
- At Least Once：通过重试机制，可以保证最终一致性，消费的幂等只需要业务消费者自行控制即可；
- Exactly Only Once：正好一次与最终一致性冲突，分布式系统中往往存在超时的问题，通过重试机制+幂等有更高的可用性；


# 消息队列的特性

In a distributed system, message queue is one of the most important component.
## 1. 异步通讯
1. Asynchronous Communication: In scenarios where the core workflow does not need to wait synchronously for non-core operations, the non-core operations can be made asynchronous which can improves the response speed of the core workflow.
## 2. 流量削峰

When a system is unable to handle massive requests or when it is not necessary to process all incoming traffic immediately, message queue can act as a buffer.

> 当一个系统不能立即处理大量的请求或不需要立即处理所有到来的流量，此时MQ可以被当作一个缓冲区

For example, if the system can handle 2000 requests per second, but during the peek time of 5000 requests per second, the system can leave 3000 requests to be stored in the MQ, preventing the system from breaking down during peek time. The requests in the MQ can be consumed by the server at a constant rate.

> 一个系统能够每秒处理2000请求，但是在流量高峰每秒有5000个请求，系统就可以将3000请求存储在MQ中，以防止宕机，在MQ中的请求可以被服务器以恒定的速率消费

## 3. 数据冗余 

When a piece of data needs to be consumed by multiple services, the message queue can distribute the message to different services. At the same time, that ensures data consistency across different parts of the system, because all modules receive the same data from the message queue.

> 当一份数据需要被多个服务消费时，消息队列可以将消息分发到不同的服务，同时也确保了数据一致性，因为所有的系统内模块都从MQ中消费相同的数据

## 4. 系统解耦

Message queues decouple applications from each other, allowing them to be developed independently. This makes systems more flexible, scalable and easier to maintain.

>MQ可以解耦系统，允许它们独立开发，这样做让系统更弹性、更具扩展性、更好维护

## 5. 并发处理 

A large task can be split into many smaller tasks, and each task can be sent as a message to the MQ, so that the tasks can be processed in parallel.

> 一个大任务可以被分割成多个小任务，每个任务都可以当成消息投递到MQ，这样可以让这些任务并发处理
## 6. 提升系统可靠性

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

