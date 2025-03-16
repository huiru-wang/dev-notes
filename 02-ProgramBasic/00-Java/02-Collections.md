
# 2个重要顶层接口

1. Collection接口
	1. List：有序、可重复集合；实现；ArrayList、LinkedList、Vector；
	2. Set：无序、不可重复集合；实现：HashSet、LinkedHashSet、TreeSet；
	3. Queue：FIFO队列；实现：PriorityQueue优先队列、ArrayQueue；
2. Map接口
	1. HashMap
	2. LinkedHashMap：维护了插入顺序；
	3. TreeMap
	4. ConcurrentHashMap：并发安全；




# List
- **ArrayList**
    - **底层结构**：动态数组，支持快速随机访问（`O(1)`）。
    - **扩容机制**：默认初始容量10，扩容时容量增加50%（如10→15→22...）。
    - **适用场景**：频繁查询，尾部插入/删除；不适用于中间频繁插入/删除（需移动元素）。
- **LinkedList**
    - **底层结构**：双向链表，每个节点保存前后引用。
    - **性能特点**：插入/删除高效（`O(1)`），随机访问慢（需遍历，`O(n)`）。
    - **适用场景**：频繁在头尾或中间插入/删除元素。
- **Vector**
    - **特点**：线程安全的动态数组（方法用`synchronized`修饰），性能较差，已逐渐被`Collections.synchronizedList`或`CopyOnWriteArrayList`取代。
# Set
- **HashSet**
    - **底层结构**：基于`HashMap`实现，Value为固定`Object`对象。
    - **性能**：插入、删除、查询平均`O(1)`，依赖哈希函数和负载因子。
    - **无序性**：遍历顺序不确定。
- **LinkedHashSet**
    - **特点**：在`HashSet`基础上，通过链表维护元素插入顺序，遍历时按插入顺序输出。
- **TreeSet**
    - **底层结构**：基于`TreeMap`（红黑树）实现，元素按自然顺序或自定义`Comparator`排序。
    - **性能**：插入、删除、查询均为`O(log n)`。

# Queue

- **普通队列**：FIFO（如`LinkedList`、`ArrayDeque`）
- **优先级队列**：按元素优先级排序（如`PriorityQueue`）
- **阻塞队列**：支持线程阻塞操作（如`BlockingQueue`实现类）
- **双端队列（Deque）**：支持两端操作（如`ArrayDeque`）




# HashMap

![](../../images/java-hashmap.jpg)

**数组 + 链表/红黑树**：当链表长度大于7，触发转换成红黑树；因为节点数大于7时，使用红黑树才有性价比；
- 使用链表：在节点数量较低，使用链表和红黑树的效率相当，代价更低；
- 使用红黑树：在节点数量较多时，使用红黑树，可以大幅提高查询效率；红黑树：自平衡 + 二分查找；

**Capacity**：哈希桶的个数；即数组的长度；构造时，设置为`2^n`个；每次扩容`* 2`
- 保持$2^n$的桶的个数，目的：可以使用**位与运算**计算key所在桶的索引；效率高于取模运算；

**Size**：HashMap中所有元素的个数；

**LoadFactor**：默认0.75f

**Threshold**：当超过阈值，触发扩容；`Threshold = Capability * LoadFactor`

**哈希算法**：计算key的哈希值；

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

## HashMap查找一个key的过程

![](../../images/java-hashmap-哈希冲突.png)

1. 计算key的哈希函数：`hash(Object key)`；
2. 计算key所在数组位置：`(length-1) & hash值;
3. 桶为空则不存在key；
4. 不为空则遍历链表/红黑树，使用对象的`equals`方法，逐步判断是否相等，直到找到 或 遍历结束；
	复杂度：最优O(1)，最坏O(n)；


## HashMap的数组长度总是$2^n$

为了加速HashMap的计算key所在桶的索引：**位与运算 效率高于 取模运算**

原本：计算元素放在哪个桶，是用Hash值对length取模得到的。但是效率太低。

现在：因为length总是2的n次方，那么 ` hash值 & (length-1)` 就可以直接得到桶的索引。提高了计算效率。

> 假设length是8（2^3）
> ` hash / 8` = ` hash / 2^3 ` = ` hash >> 3 `，同时右移挤掉的低三位，就是余数；
> ` hash & (length-1) `相当于取hash值的(length-1)个低位bit；相当于直接取余数；


## 负载因子作用

默认的负载因子=0.75，是一个经验值；**负载因子决定了扩容的阈值**；

**负载因子较低**：则阈值会远小于容量，很容易触发扩容，会使得HashMap处于比较空的状态，哈希冲突的概率会小，占用更多的空间；

**负载因子较低**：阈值会比较接近容量，不容易触发扩容；会使得HashMap处于很满的状态，容易哈希冲突，占用内存较少；


# ConcurrentHashMap
