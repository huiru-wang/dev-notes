---
title: 缓存：Redis 内存管理
category: Cache
tags:
  - cache
  - Redis
publishedAt: 2022-10-08
description: Redis的内存是如何管理的？高效的内存管理是保证内存不会溢出，服务长时间稳定运行、保持高性能的保证。
---

# Redis内存管理方式

内存资源有限，需要有一定的策略保证内存不被耗尽；

1、<font color="#de7802">过期策略</font>：Redis的每个key都可以添加过期时间，针对有过期时间的key，Redis采用定期删除 + 惰性删除的过期策略；
- 仅用于带有过期时间的键值对；
2、<font color="#de7802">内存淘汰机制</font>：过期策略并不能保证内存不被耗尽，因此仍需要在内存不足时，采取一定措施保证Redis服务可用；
- 通过配置，可以对所有的键值对进行内存淘汰；
# 过期策略

## 定期删除

定期(100ms)**随机抽取**带有过期时间的key，检查是否过期，如果过期则删除；

>存在问题：可能有key永远无法被抽取；

## 惰性删除

每当获取某个key的时候，Redis都会进行检查是否设置了过期时间，是否过期；过期则执行删除；

>存在问题：可能有key永远不再访问；

# 内存淘汰机制

过期策略存在无法解决的key，可能导致redis内存耗尽；此时需要：<font color="#de7802">内存淘汰策略</font>

内存淘汰策略需要手动开启，并指定采取的策略：

```shell
# 配置内存淘汰策略
maxmemory-policy volatile-lru
```


六种策略(3类：报错、allkeys、volatile)
- noeviction：新写入操作会报错；（不推荐）
- allkeys-lru：在键空间中，移除最近最少使用的key；（推荐）
- allkeys-random：在键空间中，随机移除某个key；（不推荐）
- allkeys-lfu：
- volatile-lru：在设置了过期时间的键空间中，移除最近最少使用的key（不推荐）
- volatile-random：在设置了过期时间的键空间中，随机移除某个key（不推荐）
- volatile-ttl：在设置了过期时间的键空间中，有更早过期时间的key优先移除；（不推荐）
- volatile-lfu：
# Lazy Free

Redis4.0之后加入后台线程，处理慢操作，并提供了lazy free机制来异步删除key的功能；

Redis 5.0版本后，LazyFree自动开启；

目的：减少了频繁的内存分配和释放；

## 配置方式

```shell
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes # 是否对过期键使用 Lazy Free 机制
lazyfree-lazy-user-del yes # 使得手动删除也采用异步删除
```

## 工作过程

1、当对象的引用计数器为 0 时，并不立即释放该对象所占用的内存，而是将该对象添加到 Redis 的一个待释放对象列表中，等待合适的时机进行释放。

2、Redis 在处理命令执行、读写数据等操作时，会在合适的时机检查待释放对象列表，逐个释放那些引用计数器为 0 的对象。这样可以将内存回收的过程和实际的命令执行解耦，减少内存回收的频率和对 Redis 性能的影响


