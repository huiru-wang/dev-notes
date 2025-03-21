---
title: DB：MySQL-InnoDB引擎及B+Tree索引
category: databse
tags:
  - databse
  - MySQL
publishedAt: 2022-12-11
description: MySQL的InnoDB是如何工作的，B+Tree索引的结构、索引的分类以及什么是覆盖索引、回表、索引下推
---
B+树索引结构：
![](/images/database-MySQL-b+tree.png)
# 为什么使用B+Tree索引

## 1. 适合做索引的数据结构特点

1. <font color="#de7802">可排序的</font>；可以利用二分搜索：AVL树、B树、B+树、B*树
2. <font color="#de7802">查询时节点尽可能的少</font>，降低IO，即多路搜索更佳：B树、B+树

## 2. 对比AVL树、B树、B+树

- AVL树、B树都存在数据量增大，<font color="#de7802">树高过高</font>、回溯查找的问题；
- B树不可范围查询；
- B+树非叶子节点全部存储索引，<font color="#de7802">能容纳更多索引</font>；
- B+树的叶子节点包含全量索引；因此每次找到叶子节点，就可以获取数据；而B树每个节点都含有数据，遍历时对每个索引需要进行中序遍历(左-根-右)的方式保证顺序；

<u>B+树目前来看是最优解</u>：

- <font color="#de7802">索引高效</font>：B+树非叶子节点只存放索引，每页能够存储更多的索引，使得索引效率很高；
- <font color="#de7802">大量减少磁盘IO</font>：大部分的表结构，只需要建立3层B+树，最多只需要三次磁盘IO；
- <font color="#de7802">有序，支持范围查找</font>：叶子节点间是双向链表连接，叶子节点中具体的记录间是单向链表；

## 3. B+树数据索引过程

1. 将<font color="#de7802">Page1</font>加载进内存中，发生一次磁盘IO；
2. 在内存中使用二分查找，确定索引列的位置，通过指针，找到下一个节点；
3. 将下一个节点加载进内存，发生一次磁盘IO；以此类推； 
4. 直到最后找到叶子节点，加载进内存，发生一次磁盘IO，并找到索引对应的数据，读取数据； 
因此三层B+树仅需三次磁盘IO，就能锁定数据；一般的表3层足矣；

## 4. B+树存储数据量计算

假设：非叶子节点只考虑 记录头和索引
- 索引列为BIGINT，占用8字节；
- 每条数据假设占用`1KB`，一页存放16条数据；
那么：
- 单页的索引数量 = 16KB / (记录头6字节 + 索引列占用大小8字节)
	一页索引量：`16KB / 14Byte = 1170 个`
- 2层索引量为：`1170 x 1170 = 1368900 个`
- 3层B+树总数据量为：`1170 x 1170 x 16 = 21,9002,400 个`
所以：
- 使用`8 byte`大小的索引，一颗B+树能够存放2000万+的索引；
- 如果<font color="#de7802">索引更大、或数据量更大</font>，则有可能构建出第4层B+Tree；
- <font color="#de7802">每多出一层，就增加IO的代价，降低索引效率</font>；


## 5. B+Tree索引特点

1. 每一个索引会构建一颗B+树，即一个索引页，每张表都最少有一个；
2. 索引页默认大小：16K，由`innodb_page_size`设置；索引页非操作系统PageCache的一页，通常为多页PageCache；
3. B+树节点包括：叶子节点、非叶子节点；
	- 非叶子节点：存储当前B+树的索引信息；
	- 叶子节点：存放具体的记录；记录间是单向链表，页间是双向链表；
4. 数据的有序性：
	- 节点内：数据按照索引列有序排序；（二分搜索查找）
	- 节点间：通过双向链表保证顺序；（保证顺序、逆序都可以遍历）
	- 但物理存储不连续，内存连续不能满足高效插入；
5. 索引的类型：
	- <font color="#de7802">聚簇索引</font>：非叶子节点存放<u>主键</u>，叶子节点存放完整数据；
	- <font color="#de7802">二级索引</font>：非叶子节点存放<u>索引列</u>，叶子节点仅存放主键，整个二级索引树来看，包含：索引列 + 主键；
# 索引分类

## 聚簇索引 Clustered Indexes

非叶子节点为主键、叶子节点包含全量数据；

- 一张表<font color="#de7802">有且只有</font>一个主键；没有显示设置主键，InnoDB会创建一个隐藏的row-id为聚簇索引
- 主键意味着全量数据的B+树；
- 新增数据时，根据主键进行插入，并维护主键索引树；
	- 当<font color="#de7802">主键递增</font>，则每次插入在树的末端，此时索引的维护代价小；
	- 当<font color="#de7802">主键乱序</font>，每次插入都可以在任何地方，当单数据页的数据过大，会发生页分裂，导致索引维护的代价很大；


## 辅助索引 Secondary Indexes

聚簇索引外的索引，统称为辅助索引；

- 非叶子节点包含：<font color="#de7802">索引列</font>；
- 叶子节点仅包含：<font color="#de7802">主键</font>；
- 如果查询结果需要索引列之外的数据，先查询到主键，再到主键B+树中查询其余数据(此过程为：<font color="#de7802">回表</font>)

### 1. 唯一索引

对某个或多个列创建唯一索引，则该列或多个列的值在整张表中保证唯一；

唯一索引<font color="#de7802">效率高</font>，因为不会有重复数据，在唯一索引B+树中，唯一的值锁定唯一的记录；

唯一索引的等值查询，在EXPLAIN中体现为：const；

### 2. 联合索引

![](/images/database-MySQL-UnionIndex.png)

联合索引结构：
- 多个索引列，以<font color="#de7802">Tuple</font>的形式存储；
- 从左向右，先以首个索引，进行排序；前一个索引相同的情况下，再以后一个索引排序；（<font color="#de7802">联合索引有最左前缀匹配原则</font>）

## 全文索引

TODO

# 索引行为

## 1. 索引覆盖

覆盖索引：<font color="#de7802">要查询所需的所有数据列，是索引数据列的子集</font>，即只需要一次索引即可返回所需数据；

覆盖索引只需一次索引，不需要回表，减少磁盘IO，效率高；

## 2. 回表

要查询的数据列，不能全部从当前的索引页中获取，需要拿到主键，再到主键索引中，获取剩余的列数据；

- 回标需要额外的IO操作；
- 回表的性能影响取决于回表的数据量；少数据量的回表无伤大雅；
- 大数据量的回表，不仅影响当前查询，还会导致缓存池脏页增多，对数据库整体性能产生影响；

## 3. 联合索引：最左前缀匹配

由于多列的联合索引的排序方式，的<font color="#de7802">只有前一个索引值确定，后一个索引才是有序</font>；
因此，联合索引的使用必须满足：最左前缀匹配原则；

假设有联合索引：a, b, c
```sql
idx_a_b_c(a, b, c)
```
那么：
1. `a = 3`；满足；可以使用到联合索引中的：a索引列；
2. `a = 1 AND b = 2`：满足；可以使用到联合索引中的：a，b索引列；
3. `b = 1 AND a = 2`：不满足；但优化器通常会自动优化为：`a = 2 AND b = 1`
4. `c = 1 AND b = 2`：不满足；且无法优化，a不确定，无法使用到b，c；
5. `a >=1 AND a <= 3 AND b = 2`：不满足，只能用到a索引列，且无法优化；a值不确定的情况下，b无法有序，则只能使用到a索引列；


## 4. 索引下推 ICP

- ICP仅用于二级索引，目的是减少回表数据量，进而减少IO；
- 对于聚簇索引，不存在回表，不会使用ICP；
- 触发索引下推，在<font color="#de7802">EXPLAIN</font>中，表现为：`using index condition`；
- 一句话：尽可能用到联合索引的所有列，将过滤操作从Server层下推到存储引擎层；以此减少回表数据量；

### 无索引下推
例如：
- 一个普通二级索引`idx_test`，包含列`(col1, col2)`；
- 主键为：id；
执行：
```sql
SELECT * FROM t 
WHERE col1 > 10 AND col1 < 20 AND col2 = 7 and col3 = 8
```

1. 通过`idx_test`的索引树，只能根据`col1`的条件来进行查询，最终遍历完`idx_text`索引后，获得的数据为：id, col1, col2

```sql
SELECT id, col1, col2 FROM t 
WHERE col1 > 10 AND col1 < 20
```

2. 根据主键id进行回表，遍历一边主键索引，获取所有满足查询出的id的数据；

```sql
SELECT id, col1, col2, col3 FROM t 
WHERE id in (xxx, xxxx)
```

3. Server层拿到所有的结果，再根据条件：`col2 = 7 and col3 = 8`，过滤出最终结果

### 开启索引下推

1. 通过`idx_test`的索引树，只能根据`col1`的条件来进行查询，最终遍历完`idx_text`索引后，获得id、col1、col2；
2. 回表前，再次根据col2的条件，进行过滤；减少回表的主键数量；
3. 回表；
4. Server层根据col3再次过滤，返回结果集；


# 索引分析

通过<font color="#de7802">EXPLAIN</font>可以显示SQL的执行计划，查看查询执行计划可以了解MySQL会如何执行某个查询，包括哪些表会被使用、使用了哪些索引以及执行顺序等信息
>重点关注：type、key、key_len、extra

id：标识每一步的执行顺序；越小越优先执行；
select_type：查询类型
- SIMPLE：简单查询
- UNION：联合查询
- SUBQUERY：子查询
- DERIVED：派生表；表示在FROM子句中使用了子查询；

table：查询涉及的表名。
partitions：匹配的分区列表。
<font color="#de7802">type</font>：访问表的方式，可以判断是否用了索引(效率从高到低)
- const：主键等值查询；如：`SELECT .. FROM id = 11`
- eq_ref：关联查询中使用：主键、唯一索引；
- ref：二级索引；包括联合索引、
- range：索引范围查找；
- index：全表扫描；但数据直接从Page中获取；
- all：全表扫描，数据从全表获取；
possible_keys：可能使用的索引列表。

<font color="#de7802">key</font>：实际使用的索引，如果是联合索引，需要结合key_len来看使用了几个索引字段；
<font color="#de7802">key_len</font>：使用的索引长度；通过使用的索引长度，可以看出联合索引中，有几个字段使用到了索引；当字段允许为null时，索引需要额外多占用1个字节；
ref：列与索引之间的匹配条件。
rows：MySQL估计需要读取多少行来执行查询。

<font color="#de7802">filtered</font>：过滤的行数占总行数的百分比。
- 如通过索引锁定了100行，根据其他WHERE条件最终返回10行，filtered=10

<font color="#de7802">Extra</font>：确定type后，一般通过此字段进一步优化；
- <font color="#95b3d7">using index</font>：走了覆盖索引；
- <font color="#95b3d7">using index condition</font>：索引下推；**InnoDB层执行过滤操作**；
- <font color="#95b3d7">using where</font>：**Server层执行了过滤操作**；InnoDB只能根据索引来进行数据过滤，如果where条件超出索引的范围，则需要由Server层进行where的数据过滤；
- using temporary：使用了临时表，需要临时存储结果集；性能低；需要优化；一般出现在：`GROUP BY、DISTINCT、UNION`；
- using filesort：排序没有走索引，需要额外的排序操作；查询出数据后，单独再进行一次`ORDER BY`排序时出现；有可能在内存排序，也有可能在磁盘排序，由：`sort_buffer_size`决定；如果数据过大，内存放不下，则会使用临时的磁盘文件进行排序；
- Backward index scan：当使用DESC排序时，索引会倒序扫描；无伤大雅；


using filesort：[order-by-optimization](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html)
using temporary：[explain-output](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

# 索引命名规范

- 主键：`pk_[column]`
- 唯一索引：`uk_[column]_[column]`
- 联合索引：`idx_[column]_[column]`

# 索引优化原则

1. <font color="#de7802">减少SQL返回的数据量</font>：只查询必要的列；禁用`SELECT *`；
	- 减少数据量，可减少网络带宽占用、增大传输效率；
	- 污染Buffer Pool：过多的数据查询，挤占Buffer Pool内存，导致其他缓存失效；
2. <font color="#de7802">分解大连接查询</font>；
	- 大的连接查询的缓存容易失效；其中一个表变化了，整个缓存失效；
	- 多个单表缓存，互相之间不影响；
	- 尽量少用连表查询：不利于分库分表、服务切分；
3. 优化索引；
	- 尽量使用索引；而非全表扫描、尽量使用覆盖索引；
	- <font color="#de7802">最小化索引构建</font>：索引数量不要太多，影响插入性能；
	- <font color="#de7802">区分度</font>：将区分度高的列尽可能放在复合索引前部，会让索引更高效；
		- 计算区分度：`count(distinct col) / count(*)`
	- <font color="#de7802">排序字段尽量使用索引字段</font>；否则需要创建临时表来获取额外的排序列；

# 索引失效场景

1. 不满足：联合索引最左前缀匹配，可能导致部分或全部索引失效；
2. 在索引列上进行计算可能导致索引失效；
	- 不走索引：`from_unixtime(create_time) = '2019-12-01'`
	- 走索引：`create_time = unix_timestamp('2019-12-01')`
3. LIKE的中缀和后缀查询索引失效；
	- 中缀：`LIKE '%xxxx%'`
	- 后缀：`LIKE '%xxxx'`
4. 列的类型转换可能会导致索引失效；
5. OR条件；
	- 参与`or`的列中，同时有索引列、非索引列，则会导致索引失效；
6. `!=`、`NOT IN` 索引失效；