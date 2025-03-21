---
title: 缓存：Redis的大key和热key的处理
category: Cache
tags:
  - cache
  - Redis
publishedAt: 2023-05-02
description: 大key和热key是分布式内存型缓存组件中比较常见的问题，对于Redis来说，如何监控、管理、紧急处理大key、热key问题很重要。
---

# 大key问题

大key的大小视实际业务体量、Redis资源大小而定，可以是：
- value值占用内存过大的key；
- list成员过多；
- hash类型field过多；
## 影响

1. 当对大Key执行读写时，CPU读写变慢，并占用网络带宽、传输耗时，影响性能;
2. 集群下可能数据倾斜，主从同步可能延迟;
3. 持久化策略会更加耗时，如果是AOF同步写，会阻塞主进程;
4. 如果`bgsave`、`bgwriteaof`时触发了写大key，会触发内存复制，阻塞主进程，并加剧内存的占用

## 排查措施

1. <font color="#de7802">bigkey命令</font>：列出string类型中内存占用最大的那个key，set、list、zset、hash列出元素最多的那个key；
```shell
# --i 参数，降低扫描的执行速度；0.1 表示 100 毫秒执行一次，防止阻塞进程；
./redis-cli --bigkeys --i 0.1
```
2. <font color="#de7802">分析RDB文件</font>，可以做到离线分析，不影响进程；但是分析速度会比较慢；
3. **redis-rdb-tools工具(python脚本)**
	- 可以找到大于指定大小的key；

## 预防

1. 做好内存、CPU监控；
2. 业务层面把控Redis键值对的存储粒度，控制key的大小；设置合理过期时间；
3. 开启Lazy Free的内存淘汰、过期等删除机制；保证触发时，不影响业务；

## 大key的删除策略
### 1. 手动异步删除

`unlink`：直接异步删除；需要开启：[lazy-free](./02_Redis-内存管理.md#lazy-free)
`del + lazyfree_lazy_user_del 配置项开启`：异步删除；

### 2. 集合结构分批删除

对于集合类的key，可以根据集合特点，循环分批或采用一定策略删除部分或全部数据：

1、分批删除全部的数据；
2、根据业务特点，分批删除不需要的部分数据，如：
- zset中删除比重小于阈值的数据；
- hash数据中删除指定的较大的field；

#  热key问题

<font color="#de7802">热key</font>：QPS、带宽、CPU集中在特定的Key；

1、<font color="#de7802">对不需要写的热点key</font>，可以加载到服务节点内存中(二级缓存)，减少请求链路，降低Redis的压力；

2、集群场景下，热key可能对某个单节点压力较大，可以业务上对热key进行二分分片(如增加一个分片字段)，路由到不同的分片节点，分摊压力；



