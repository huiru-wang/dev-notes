---
title: 缓存：Redis基本学习
category: Cache
tags:
  - cache
  - Redis
publishedAt: 2022-09-20
description: Redis因为其In-memory、数据结构、单线程、高效的IO模型，拥有非常好的性能。
---

# Why is Redis so fast

## 1. In-memory Storage

Redis stores data <font color="#de7802">in memory</font>.

Memory access is much faster than disk I/O. Reading and writing data from/to memory can be completed in a very short time

> Redis将数据存储在内存中
> 内存的访问速度要比磁盘快的多，读写内存数据只需要很短的时间

## 2. Efficient Data Structures

Redis has <font color="#de7802">a variety of efficient data structures</font> such as strings, hashes, lists, sets, and sorted sets. These data structures are implemented with highly optimized algorithms.

> Redis有丰富的高效数据结构，如strings、hash表、list、set和zset；

## 3.  Single-Threaded Read/Write

In Redis, the <font color="#de7802">single-threaded model</font> simplifies the code structure and reduces the overhead of context switching between threads. There is no need to deal with complex issues such as thread synchronization and locking. Each command can be executed quickly and sequentially.

> 在Redis中，单线程模型简化了代码结构，减少了线程间上下文切换的开销，无需处理线程同步、锁定等复杂问题，各个命令可以快速、顺序地执行。

## 4. Non-Blocking IO Multiplexing

Redis uses <font color="#de7802">non-blocking I/O multiplexing mechanisms</font>. This allows Redis to handle multiple client connections efficiently. Redis doesn't block waiting for the I/O operation to complete. Instead, it can continue to handle other requests or perform other tasks.
When the I/O operation is ready, Redis will be notified and then process the relevant I/O event immediately.

> Redis 使用非阻塞 I/O 多路复用机制。这使 Redis 能够高效地处理多个客户端连接。Redis 不会阻塞等待 I/O 操作完成。相反，它可以继续处理其他请求或执行其他任务
> 当 I/O 操作准备就绪时，Redis 将收到通知，然后立即处理相关的 I/O 事件。

# Redis Datastructure and Command

## 1. string

String data structure in Redis is a simple key-value pair. It can be used to store integer,  text string and Json string.

Commnad:
- `SET`/`GET`: used to set / retrieve a key-value pair. If the value does not exsit, `nil` is returnd.
- `MSET` / `MGET`: set / get multiple key-value pair in one operation in the order they are requested. If the value does not exsit, `nil` is returnd for that key's position.
- `INCR` / `DECR`: increment / decrement the value of a key that stores an integer.

## 2. Hash

A hash data structure in Redis is a collection of key-value pairs. It is like a dictionary or a map in programming languages.
It is useful for storing objects with multiple fields.

Command:
- `HSET` / `HGET`: `HSET` is used to set a field-value pair inside a hash. `HGET` retrieves the value of a specific field in a hash.
- `HMSET` / `HMGET`: allows you to set multiple field-value pair and retrieves values of multiple fields.
- `HDEL`: used to delete one or more fields from a hash.

## 3. List

A list data structure in Redis is an ordered collection of strings. 
It can be used to store a sequence of values.
The elements in a list are ordered, and you can access them by index.

Command:
- `LPUSH` / `RPUSH`: allow you push one or more elements to the left / right of a list.
- `LPOP` / `RPOP`: allow you pop one or more elements from the left / right of a list.
- `LRANGE`: retrieves a range of elements from a list.

## 4. Set

A set in Redis is an unordered collection of unique strings. 
It is used to store a group of distinct values.
It can store `user Ids`, `tags` which the order of elements in a set does not matter, and duplicate elements are not allowed.

Command:
- `SADD` / `SMEMBERS`: `SADD` adds one or more members to a set. and `SMEMBERS` can retrieve all the elements of a set.
- `SREM`: removes member from a set. `SREM myset "member1"`
- `SISMEMBER`: checks if a given value is a member of a set. 

## 5. Zset(Sorted Set)

A sorted set in Redis is a collection of unique members, each associated with a score.
The members are stored based on their scores in ascending or descending order.
It is useful for ranking systems.

Command:
- `ZADD`: adds one or more members with their scores to a sorted set. `ZADD myzset 1 "member"` adds "member" with a score of 1 to `myzset`.
- `ZRANGE`: retrieves a range of members from a zset by scores. `ZRANGE 0-30` returns all the elements of the zset.
- `ZREM`: removes one or more members from a zset.
- `ZRANK` / : get the rank of a member. `ZRANK myzset "member1"` returns the position of `"member1"` in the `myzset` sorted set.
- `ZREVRANK`: get the descending order of score.

## Common Commnad

- `EXISTS key [key1, key2]`: checks if one or more keys exist.
- `DEL key [key1, key2]`: deletes one or more keys.
