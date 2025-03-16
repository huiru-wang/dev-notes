---
title: 分布式系统设计基础
categorystem: 分布式系统
tags:
  - cap
  - systemDesign
publishedAt: 2023-03-12
description: 介绍CAP理论和分布式系统关键系统衡量指标
---

# 单点服务和分布式服务

## 单点服务
1. Simplicity
In a signle-server system, the overall system architecture is relatively simple. All services, applications and data are stored on a single server.
And it is easy to develop and deploy applications to a single server. 
Compared with distributed systems, there is no need to master complex distributed framework and middleware.
2. Cost-effectiveness: Low hardware cost and Low maintenance cost.
A single server only requires one server device which can save a lot of hardware cost.
It is more simple to maintaning a single server.
3. Easy Performance Optimization
In a single server, all resources such as CPU, memory, storage are concentrated on one device. It is more conveniently monitor and allocate resources to optimize system performance.
In distributed services, nodes usually need to communicate through RPC, which will more or less cause some delay.
4. Single Point of Failure
The biggest problem of the single server system is the single-point-of-failure. if this server has a hardware failure, the entire system will stop running and the data may be lost.
5. Limited Scalability
When the business grows, such as an increase in website traffic and an increase in the data volume and processing requirements of applications, a single server may not be able to meet the performance requirements.
And it is difficult to deal with large scale concurrent requests and complex business logic expansiion
## 分布式服务

1. High Availability and Fault-Tolerance
Distributed systems provide redundancy by storing data and running services on multiple nodes.
And the failure of an individual node does not lead to the collapse of the entire system.

2. Scalability
Distributed systems' processing power and storage capacity can be increased by adding more nodes. The new nodes added can automatically participate in data processing tasks and share the load.
It is relatively flexible and not limited by hardware resources like a single server system.

3. Performance Improvement
Distributed systems offen use load balancers to evently distribute client requests to multiple servers to improve the performance of the overall service.

4. Complexity
Distributed systems usually have a complex architecture and rely on more distributed middleware such as distributed message queue, distributed cache service.

5. Date-Consistency Chanllenges
It is difficult to achieve strong data consistency in a distributed system. It is difficult for all nodes see exactly the same data at the same time.

6. Network Communication Problems
Distributed systems rely heavily on networks for communication among nodes. Network delays and network failures will all affect the systems' performance and stability, and may even cause services to fail.

# 1. CAP

It is not about finding the perfect solution 
It is about finding the best solution for our specific use case.

<span style="background:#d2cbff">Partition tolerance is fundamental in distributed systems because network partitions are almost inevitable. </span>
So when P is reality, system designers are faced with a trade-off between Consistency and Availability.

## 1.1 Partition Tolerance

Partition tolerance refers to the ability of a distributed system to continue operating despite network partitions.

In actual distributed systems, partition tolerance (P) is often inevitable.

![](/images/systemDesign-CAP-Partition.png)

## 1.2 Consistency

Consistency ensures all nodes in a distributed system have the same data at the same time.

>When a write operation is performed on one node, the effect of that write is immediately visible to all other nodes when a read operation is subsequently performed.

For example, in a database cluster, if a record's value is updated in one database server, any other server in the cluster that accesses that record should return the updated value.

![](/images/systemDesign-CAP-Consistency.png)

## 1.3 Availability

In a distributed system, availability means every request received by a non-failing node in the system must result in a response within a reasonable time.

> When we talk about availability we are essentially asking is our system up and running when our users need it. Even if some servers in the cluster fail, the remaining healthy servers should be able to handle the requests and provide the service.

$$
Availability \ Requirement \ Metric=\frac{Total \ Time-Down \ Time}{Total \ Time} \ast 100\%
$$

99.9% Availability => 8.76 hours of downtime
99.99% Availability => 52.6 minutes of downtime
99.999% Availability => 5 minutes of downtime
![](/images/systemDesign-CAP-Availability.png)

## 1.4 Why only AP or CP

<font color="#de7802">CP System</font>: a system emphasizes consistency. These systems need to ensure that all nodes have the same view of data.
- MySQL
- Distributed File System. Like Hadoop
- ETCD, Zookeeper

<font color="#de7802">AP System</font>: a system prioritize availability. These systems are designed to always provide a response to requests.
- NoSQL Database. Like MongoDB, Redis
- CDN

# 2. SLO & SLA

SLO: Service Level Objectives. 

SLA: Service Level Agreements. SLA is an agreement between a service provider and a customer regarding measurable metrics such as uptime, responsiveness and liability.
>SLA（服务级别协议）是提供商和客户之间关于正常运行时间、响应能力和责任等可衡量指标的协议。
# 3. Key Metric

## 3.1 Reliability (可靠性)

#TODO 


## 3.2 Scalability (可扩展性)

Scalability refers to the system's ability to handle an increasing amount of work by adding resources, like servers, storage.

If the system is scalable, it can easily add more servers to handle the increased traffic.

<font color="#de7802">Throughput</font> is one way to measure scalability of a system. Throughput is the number of transactions or operations the system can handle per unit of time.

## 3.3 Consistency (一致性)

Consistency refers to all nodes of the system have the same view of data at a given point in time.

Read-write test is one way to measure the consistency of a system.

## 3.4 Durability (持久性)

Durability refers to the system's ability to ensure that data is not lost even in the face of various failures. Data is typically stored in a persistent storage. Like hard drive, distributed storage systems.

MTDBL and RPO are two key metrics to measure a system's durability.

MTDBL:  the mean time between data losses. A high-quality distributed system would aim for a very long MTDBL.
RPO: the maximum amount of data that a system can afford to lose. The closer the RPO is to zero, the more durable the system.

## 3.5 Performance (性能指标)

Better performance usually means it can handle requests faster and support more concurrent users.

<font color="#de7802">Throughput</font>: how much data the system can handle per unit of time. It is a high-level concept. In different systems, there are more precise metrics to describe Throughput.

- RPS: Requests Per Second. Web service Throughput.
- QPS: Query Per Second. DB Throughput. 
- B/s: Bytes Per Second. Kafka Throughput.
- TPS: Transaction Per Second.

<font color="#de7802">Latency</font>: how long it takes the system handle a single request.

- RT: Response Time. RT refers to the time interval from when a client sends a request to when it receives a response from the service provider.

>Optimizing for one can often lead to sacrifices in the other. For example, Batching operations can increase Throughput, but might also increasing Latency.



