

# MySQL常见问题

## 1.为什么 Innodb 用 「B+」树作为索引？



## 2.MySQL版本特性

1. MySQL 默认的存储引擎在「5.5.5」版本之前是 MyISAM，在「5.5.5」版本及之后的版本都是 InnoDB；
2. MySQL 的 binlog 在「3.23 - 5.0.0」版本中只支持 `基于 SQL 语句（STATEMENT）` 的格式，从「5.1」版本开始支持 `基于数据行` 的格式；
2. MySQL 8.0 版本直接将查询缓存的整块功能删掉了，也就是说 8.0 开始彻底没有这个功能了；

## 3.InnoDB和MyISAM的区别

目前广泛使用的是MyISAM和InnoDB两种引擎：

1.MyISAM引擎是MySQL 5.5.5之前版本的默认引擎，它的特点是：

- 不支持行锁，读取时对需要读到的所有表加锁，写入时则对表加排它锁
- 不支持事务
- 不支持外键
- 不支持崩溃后的安全恢复
- 在表有读取查询的同时，支持往表中插入新纪录
- 支持`BLOB`和`TEXT`的前500个字符索引，支持全文索引
- 支持延迟更新索引，极大提升写入性能
- 对于不会进行修改的表，支持压缩表，极大减少磁盘空间占用

2.InnoDB在MySQL 5.5后成为默认索引，它的特点是：

- 支持行锁，采用MVCC来支持高并发
- 支持事务
- 支持外键
- 支持崩溃后的安全恢复
- 不支持全文索引

总体来讲，MyISAM适合`SELECT`密集型的表，而InnoDB适合`INSERT`和`UPDATE`密集型的表

## 4.最左索引匹配规则

顾名思义就是最左优先，在创建组合索引时，要根据业务需求，where子句中使用最频繁的一列放在最左边。复合索引很重要的问题是如何安排列的顺序，比如where后面用到c1, c2 这两个字段，那么索引的顺序是(c1,c2)还是(c2,c1)呢，正确的做法是，重复值越少的越放前面，比如一个列 95%的值都不重复，那么一般可以将这个列放最前面

- 复合索引index(a,b,c)
- where a=3 只使用了a
- where a=3 and b=5 使用了a,b
- where a=3 and b=5 and c=4 使用了a,b,c
- where b=3 or where c=4 没有使用索引
- where a=3 and c=4 仅使用了 a
- where a=3 and b>10 and c=7 使用了a,b
- where a=3 and b like 'xx%' and c=7 使用了a,b
- 其实相当于创建了多个索引：key(a)、key(a,b)、key(a,b,c)
