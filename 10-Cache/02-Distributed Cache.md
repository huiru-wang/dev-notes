---
title: 分布式缓存组件
category: Cache
tags:
  - cache
publishedAt: 2022-08-02
description: 分布式缓存组件解决了什么问题？架构是什么样的？有什么痛点和问题？
---

# 单体架构

早期的单节点服务，如在进程内的HashMap，仅服务于单机，无法跨进程共享数据，服务实例间的缓存冗余，扩展性差；

方案：引入集中式缓存组件；独立部署缓存服务节点。

单节点的分布式缓存（集中式缓存）：
1. 有单点故障问题；
2. 内存有限，扩容困难；


# 主从复制和故障转移

主从复制是解决<font color="#f79646">单点故障</font>而出现；传统数据库、Redis都需要从节点在主节点宕机时随时顶上。

但是光有从节点只是做了数据的备份，如何检测主节点服务状态、如何进行故障转移？

Redis的解决方案有2个：
- Sentinel方案；
- Redis Cluster方案；

# 分片技术

分片技术是Redis为了解决单节点内存容量有限的问题，通过扩展节点个数来实现数据分片，提高集群的承载能力；


# 集群技术

当前Redis的集群架构大致有2种：
- Redis的开源集群方案；（客户端直连Redis节点，客户端进行命令路由、集群内借助Gossip协议、一主多备进行故障转移）
- 代理模式方案；（客户端只和代理节点交互，由代理进行命令的路由、负载均衡、故障转移）


  
Distributed caching is a technique used in a distributed system to store and manage data in a way that allows multiple nodes or servers in a distributed environment to access and share cached data.

> 分布式缓存是分布式系统中使用的一种技术，用于存储和管理数据，允许分布式环境中的多个节点或服务器访问和共享缓存数据。

# Benefits

## 1. Performance Improvement

  
## 2. Scalability
  

## 3. High Avaliability



