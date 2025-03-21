---
title: DB：MySQL语句的执行过程
category: databse
tags:
  - databse
  - MySQL
publishedAt: 2023-02-13
description: MySQL中的 Select/Update 如何工作，在架构中的执行流程，各个关键字的执行顺序和方式
---

# SELECT语句

1. Server层：客户端发送SQL到Server层：
	- 连接器检查权限
	- 解析器完成语法分析
	- 优化器根据IO、CPU成本，生成最优执行计划；选择合适的索引
	- 由执行器调用存储引擎的接口执行SQL（通常是InnoDB）；
2. InnoDB存储引擎层：InnoDB根据SQL使用到的索引，读取对应的页；
	- 如果<font color="#548dd4">索引页</font>在<font color="#de7802">Buffer Pool</font>中，则直接读取使用；
	- 如果<font color="#548dd4">索引页</font>不在<font color="#de7802">Buffer Pool</font>中，则从磁盘读取，加载到内存，再执行查询；
3. 普通的查询语句为快照读，根据使用的隔离级别，创建对应的ReacView读视图：
	- InnoDB会给当前事务分配事务id，并根据当前活跃事务和要查询的数据行，获取可以读到的数据版本；
4. 根据where条件筛选数据：（InnoDB只能根据索引列筛选）
	- 如果where条件的列可以通过索引进行筛选，则在InnoDB完成筛选；
	- 如果where条件的列超出索引列，则需要在Server层再次筛选过滤；
5. 根据查询的列，读取对应的数据；
	- 如果查询的列范围小于当前的索引树，则直接获取结果集；（覆盖索引）
	- 如果查询的列范围大于当前的索引树，则触发回表，拿到主键id，再通过主键索引树，获取其余的列值；（回表）
6. 如果有Order By排序：
	- 走索引：order by的列和索引一致，则效率最高，可以直接根据索引排序，避免额外的排序；（前提有where条件，根据查询条件，再结合联合索引原理，分析是否走索引排序）
	- 不走索引，要看数据量和分配给查询线程的`sort_buffer_size`的大小，如果小于排序缓存区，则可以直接在内存中排序；
	- 不走索引，且数据量比较大，或设计多表连接，通常会触发临时文件排序；
7. 如果有limit：根据limit的值，取出最终的列数；
8. 返回Server结果集，在返回客户端

# UPDATE语句

1. Server层：客户端发送SQL到Server层
	1. 由连接器检查权限
	2. 由解析器完成语法分析
	3. 优化器选择最优执行计划；
	4. 由执行器调用Innodb执行SQL；
2. InnoDB层：开启事务，分配事务id，先执行查找，后执行更新
3. update为当前读，需要读取最新的数据而非快照数据：
	- 如果数据在Buffer Pool中，则直接读取；
	- 如果数据不在Buffer Pool中，则需要从磁盘加载；
	- 如果能走索引：根据索引查找到要修改的数据行，尝试独占的方式获取行锁；
	- 如果不走索引：尝试获取表锁；
	- 如果无法获取锁，则等待，直到锁被释放；
4. 拿到锁后，执行数据更新：
	- 将要修改的原始记录写入`undo log`中；（支持回滚和MVCC数据版本）
	- 将要更新的数据写入Buffer Pool的数据页中，并标记页为脏页；
5. 准备提交阶段：
	- 执行器将数据修改写入`redo log buffer`中；（根据配置择时刷盘）
	- 执行器将SQL操作写入`bin log`
	- 在`redo log`中标记事务为准备提交状态；
6. 提交阶段：当事务结束，执行commit
	- 将`redo log`标记为已提交状态；
	- 提交事务、释放锁；
7. 脏页数据会放入Flush链中，由异步线程完成刷盘操作；

`redo log buffer`根据配置`innodb_flush_log_at_trx_commit`择时刷盘：
- `innodb_flush_log_at_trx_commit = 0`：写入buffer，每秒异步写入磁盘；(性能高，宕机可能丢数据)
- `innodb_flush_log_at_trx_commit = 1`：写入buffer，同步写入磁盘；(默认值；可靠性高，保证不丢失已提交数据)
- `innodb_flush_log_at_trx_commit = 2`：写入buffer，由操作系统异步落盘；(性能高，可能丢数据)

# SQL关键字执行顺序

1. **FROM**：确定主表；
2. **JOIN ON**：连接子表，并通过ON条件，将子表关联的数据，插入主表；
3. **WHERE**：根据查询条件，对数据过滤；
4. **GROUP BY**：对过滤后的结果集，进行分组；
5. **HAVING**：对分组后的数据，执行聚合函数，再次过滤；
6. **SELECT**：从结果中查询指定的列；
7. **DISTINCT**：如果有，再对查询的结果按列去重；
8. **ORDER BY**：对最终的结果，排序；
9. **LIMIT**

# JOIN是如何工作的

```sql
SELECT <column...>
FROM <table_1> t1
JOIN <table_2> t2 ON <join_condition>
JOIN <table_3> t3 ON <join_condition>
WHERE <where_condition>
LIMIT 10;
```

- 无论多少JOIN，都又一个表驱动，具体看JOIN方式；

1、确定驱动表：
0、INNER JOIN中ON是没有用的，同`where`效果一样；是先连接
# GROUP BY如何工作

<font color="#de7802">group by</font>的本质是排序，只有使用了索引的情况下，才有可能不借助临时表就能完成group操作；否则需要在内存中完成group，并且如果数据量过大，还可能要借助磁盘文件；
- `group by`的列必须是同一个表的，且必须在同一个索引内，才有可能走索引；具体还需要看具体的聚合函数；
- `group by`不走索引，则会使用临时表，但是数据量过大，仍然需要借助磁盘文件，由：`tmp_table_size` 决定；

`group by`两种执行方式：
- 松索引扫描
- 紧索引扫描

# ORDER BY如何工作

1、初始化`sort_buffer`，根据查询的结果集，将`select`的列放入buffer中；(`order by`一般在sql最后执行)
2、有以下两种情况时，在将数据放入buffer中时，就已经排好序了：
- **使用了单列索引，并且`where`和`order by`使用同一单列索引；where走了索引天然有序，但是如果用另一个字段排序，则需要重新排序**；
- **使用了联合索引，且`order by`的是索引的最后一列**；
3、如果explain出现：`using filesort`，一般是以下情况：
- 没走索引进行排序，需要额外进行一次快速排序；
- 即使使用了索引，如果数据量过大，也可能需要使用临时的磁盘文件进行排序；由：`sort_buffer_size`决定；

# in / exist

in执行流程：查询子查询的表且内外表有关联时，先执行内层表的子查询，然后将内表和外表做一个笛卡尔积，然后按照条件进行筛选，得到结果集。
- 所以相对内表比较小的时候，in的速度较快

exists执行流程：指定一个子查询，检测行的存在。遍历循环外表，然后看外表中的记录有没有和内表的数据一样的，匹配上就将结果放入结果集中
因此：
- 遵循小表驱动大表；
- `exists`是以外层表为驱动表，`in`是先执行内层表的；
- 如果子查询得出的结果集记录较少，主查询中的表较大且又有索引时应该用`in`；
- 如果外层的主查询记录较少，子查询中的表大且又有索引时使用`exists`

## not in / not exists

`not in`使用的是全表扫描没有用到索引；
`not exists`在子查询依然能用到表上的索引；
- 建议使用`not exists`代替`not in`；
