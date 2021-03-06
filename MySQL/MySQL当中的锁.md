# MySQL当中的锁

## 1.InnoDB的加锁逻辑

**事务隔离级别为读已提交 `RC` 时**

1. 当 `WHERE` 条件列为主键时，将对对应的聚簇索引记录加 `X` 锁；
2. 当 `WHERE` 条件列为唯一索引时，将对对应索引值的记录和关联的聚簇索引记录加 `X` 锁；
3. 当 `WHERE` 条件列为非唯一索引时，将对对应索引值的多条记录和关联的多条聚簇索引记录加 `X` 锁；
4. 当 `WHERE` 条件列为没有索引时，由于查询列上没有索引，因此只能走聚簇索引，进行全表扫描，这使得聚簇索引上所有的记录都被加上了 `X` 锁（锁住的是聚簇索引记录）；在通过主键列查询时，若以没有索引的列对应记录的列值以外的值作查询，则可以请求到 `X ` 锁（MySQL 有一些改进，在 MySQL Server 过滤条件，发现不满足后，会调用 `unlock_row` 方法，把不满足条件的记录放锁）。

**事务隔离级别为可重复读 `RR` 时**

1. 当 `WHERE` 条件列为主键时，将对对应的聚簇索引记录加 `X` 锁；
2. 当 `WHERE` 条件列为唯一索引时，将对对应索引值的记录和关联的聚簇索引记录加 `X` 锁；
3. 当 `WHERE` 条件列为非唯一索引时，将对对应索引值的多条记录和关联的多条聚簇索引记录加 `X` 锁；此外，为了防止发生幻读，将对对应索引值记录附近的间隙加间隙锁，会阻塞其它事务对索引值记录附近的间隙的插入与删除操作；
4. 当 `WHERE` 条件列为没有索引时，由于查询列上没有索引，因此只能走聚簇索引，进行全表扫描，这使得聚簇索引上所有的记录都被加上了 `X` 锁（锁住的是聚簇索引记录）；在通过主键列查询时，若以没有索引的列对应记录的列值以外的值作查询，则可以请求到 `X ` 锁（MySQL 有一些改进，在 MySQL Server 过滤条件，发现不满足后，会调用 `unlock_row` 方法，把不满足条件的记录放锁）；此外，为了防止发生幻读，将对对应索引值记录附近的间隙加间隙锁，会阻塞其它事务对索引值记录附近的间隙的插入与删除操作。

> **可重复读 `RR`** 隔离级别在 `WHERE` 条件列为「非唯一索引和没有索引」时，比**读已提交 `RC` **隔离级别多了在索引值记录附近间隙上加的「间隙锁」，这也正是**可重复读 `RR`** 隔离级别不会出现幻读的原因。

**事务隔离级别为可序列化 `Serializable` 时**

可序列化 `Serializable` 作为最高隔离级别，在执行「插入、更新、删除」这样的**当前读**操作时，与可重复读 `RR` 隔离级别一致。只是在执行没有加锁「查询」的**快照读**操作时，可序列化 `Serializable` 隔离级别将对记录加上共享锁。因此，可序列化 `Serializable` 隔离级别下，不存在「MVCC」式的快照读。

## 2.MySQL当中的锁

### 1.共享锁

允许持有该锁的事务读取某一行记录。持有共享锁的事务允许其他事务读取该行记录，并且获取到该记录的共享锁，但不允许其他事务对该记录进行更新或删除。

### 2.排它锁

允许持有该锁的事务更新或删除某一行。持有排它锁的事务不允许其他事务对同一行记录加锁。

### 3.意向锁

意向锁属于表级锁，用来表明事务想要在表中的行上获取什么类型的锁（共享或独占），不同的事务可以在同一个表上获取不同类型的意向锁，但是第一个事务获取表上的意向排它锁(IX)可以阻塞其他事务获取该表上的任何S或X锁。相反，获取表上的意向共享锁(IS)的第一个事务将阻塞其他事务获取表上的任何X锁。两阶段过程允许按顺序解决锁定请求，而不阻塞锁定和对应的兼容操作。

InnoDB这样定义了上述两种意向锁：

- 意向共享锁(IS)：事务T尝试在表中某些行上设置共享锁(S)，如使用 SELECT ... LOCK IN SHARE MODE；
- 意向排它锁(IX)：事务T尝试在表中某些行上设置排它锁(X)，如使用SELECT ... FOR UPDATE。

意向锁同时需要遵循以下协议：

- 在事务可以获取表t中的行的S锁之前，它必须首先在t上获取IS或更强的锁；
- 在事务可以获取行上的X锁之前，它必须首先在t上获取IX锁。

### 4.记录锁

记录锁针对的是索引记录，对表中的记录加锁，叫做记录锁，简称行锁。

### 5.间隙锁

间隙锁是一种位于索引记录间锁间的锁(包括位于第一条索引记录之前，或最后一条索引记录之后的间隙)，而不锁住记录本身。

根据检索条件向左寻找最靠近检索条件的记录值A，作为左区间，向右寻找最靠近检索条件的记录值B作为右区间，即锁定的间隙为（A，B）。

**间隙锁的目的是为了防止幻读，其主要通过两个方面实现这个目的：**

1. 防止间隙内有新数据被插入；
2. 防止间隙内有新数据被删除；
3. 防止已存在的数据，更新成间隙内的数据。

**InnoDB自动使用间隙锁的条件：**

1. 必须在 RR 级别下；
2. 检索条件必须有索引。

### 6.Next-Key锁

Next-Key Lock 是记录锁和间隙锁的组合，锁定记录本身且锁定范围。主要目的是解决幻读的问题。

### 7.自增锁



### 8.表锁



