---
title: "Mysql实战45讲笔记"
date: 2021-04-25T09:47:34+08:00
tags: ["mysql"]
draft: true
---

- Mysql学习

<!--more-->

## 01讲 基础架构：一条sql查询语句是如何运行的

### mysql基本架构

![mysql基础架构](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/mysql.png)

### mysql可以分为Server层和存储引擎层两部分。

- Server层包括连接器、查询缓存、分析器、优化器、执行器
- 存储引擎层负责数据的存储和提取。插件式，支持InnoDB、MyISAM、Memory等

#### 连接器

- 负责跟客户端建立连接、获取权限、维持和管理连接
- `mysql -h$ip -P$port -u$user -p`
- 客户端长时间无操作会被连接器自动断开。由参数wait_timeout控制。默认8小时
- 通过执行 mysql_reset_connection来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。

#### 查询缓存

- 将参数query_cache_type设置成DEMAND，这样对于默认的SQL语句都不使用查询缓存。
- 对于确定要使用查询缓存的语句，可以用SQL_CACHE显式指定：`select SQL_CACHE * from T where ID=10；`
- MySQL 8.0版本直接将查询缓存的整块功能删掉了，也就是说8.0开始彻底没有这个功能了。

#### 分析器

- 对SQL语句先做词法分析，然后做语法分析。

#### 优化器

- 决定使用哪个索引
- 决定表关联连接顺序

#### 执行器

- 首先判断用户执行权限
- 慢查询日志中rows_examined字段，表示这个语句扫描了多少行。这个值就是在执行器每次调用引擎获取数据行的时候累加的。



## 02讲 日志系统：一条SQL更新语句是如何执行的

### 日志模块：redo log

- InooDB引擎特有的日志
- WAL技术，全称是Write-Ahead Logging，关键点就是先写日志，再写磁盘
- 有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为**crash-safe**。
- innodb_flush_log_at_trx_commit这个参数设置成1的时候，表示每次事务的redo log都直接持久化到磁盘。这个参数我建议你设置成1，这样可以保证MySQL异常重启之后数据不丢失。

### 重要的日志模块：binlog

- Server层日志

- redo log和binlog区别
  1. redo log是InnoDB引擎特有的；binlog是MySQL的Server层实现的，所有引擎都可以使用。
  2. redo log是物理日志，记录的是“在某个数据页上做了什么修改”；binlog是逻辑日志，记录的是这个语句的原始逻辑，比如“给ID=2这一行的c字段加1 ”。
  3. redo log是循环写的，空间固定会用完；binlog是可以追加写入的。“追加写”是指binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。
- sync_binlog这个参数设置成1的时候，表示每次事务的binlog都持久化到磁盘。这个参数我也建议你设置成1，这样可以保证MySQL异常重启之后binlog不丢失。



## 03讲 事务隔离：为什么你改了我还看不见

### 隔离性与隔离级别

- ACID（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性）
- 为了解决脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题，于是有了“隔离级别”的概念。

#### 隔离级别

- 读未提交（read uncommitted）：一个事务还没提交时，它做的变更就能被别的事务看到。
- 读提交（read committed）：一个事务提交之后，他做的变更才会被其他事务看到。
- 可重复读（repeatable read)：一个事务执行过程中看到的数据，总是跟这个事务在启动时候看到的数据是一致的。在可重复读隔离级别下，为提交变更对其他事务也是不可见的。
- 串行化（serializable）：对同一行记录，读写都会加锁。当出现锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

#### 事务隔离的实现

#### 事务的启动方式

- 显式启动事务语句， begin 或 start transaction。配套的提交语句是commit，回滚语句是rollback。
- set autocommit=0，这个命令会将这个线程的自动提交关掉。意味着如果你只执行一个select语句，这个事务就启动了，而且并不会自动提交。这个事务持续存在直到你主动执行commit 或 rollback 语句，或者断开连接。

查找持续时间超过60S的事务：

`select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60`



## 04讲 深入浅出索引（上）

索引的出现就是为了提高查询的效率，类似书的目录一样。

### 索引的常见模型

- 哈希表、有序数组和搜索树
  - 哈希表是一种以键-值（key-value）存储数据的结构，我们只要输入待查找的值即key，就可以找到其对应的值即Value。
    - 哈希表这种结构适用于只有等值查询的场景，比如Memcached及其他一些NoSQL引擎。
  - 有序数组在等值查询和范围查询场景中的性能就都非常优秀
    - 有序数组索引只适用于静态存储引擎，在需要更新数据的时候比较麻烦

### InnoDB索引模型

在InnoDB中，表都是根据主键顺序以索引的形式存放的，这种存储方式的表成为索引组织表。InnoDB使用了B+树索引模型，所以数据都是存储在B+树中。

每一个索引在InnoDB中对应一棵B+树。

索引类型分为`主键索引`和`非主键索引`：

- 主键索引的叶子节点存的是整行数据。InnoDB中，主键索引也被称为`聚簇索引`。
- 非主键索引的叶子节点存的是主键的值。InnoDB中，非主键索引也被称为`二级索引`。
- 基于主键索引和普通索引的查询有什么区别？
  - 主键查询方式，则只需要搜索ID这棵B+树；
  - 普通索引查询方式，则需要先搜索k索引树，得到ID的值为500，再到ID索引树搜索一次。这个过程称为回表。
  - 基于非主键索引的查询需要多扫描一棵索引树。因此，我们在应用中应该尽量使用主键查询。

### 索引维护

重建索引`alter table T engine=InnoDB`



## 05讲 深入浅出索引（下）

回到主键索引树搜索的过程，我们称为回表。

### 覆盖索引

在这个查询里面，索引k已经“覆盖了”我们的查询需求，我们称为`覆盖索引`。

- select ID from T where k between 3 and 5，这时ID的值已经在k索引树上了

### 最左前缀原则

在建立联合索引的时候，如何安排索引内的字段顺序。

- 第一原则是，如果通过调整顺序，可以少维护一个索引，那么这个顺序往往就是需要优先考虑采用的。
- 然后考虑空间。

### 索引下推

可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。

`select * from tuser where name like '张%' and age=10 and ismale=1;`，根据索引搜索的时候，InnoDB在(name,age)索引内部就判断了age，对于不符合的记录，直接判断并跳过。只需要对ID4、ID5这两条记录回表取数据判断，就只需要回表2次。

## 06讲 全局锁和表锁

根据加锁的范围，Mysql的锁可以分为全局锁、表级锁、行锁

### 全局锁

对整个数据库实例加锁，一般用于全库逻辑备份。

MySQL提供了一个加全局读锁的方法，命令是 Flush tables with read lock (FTWRL)。当你需要让整个库处于只读状态的时候，

对于支持事务的引擎，可以通过事务，保证拿到一致性视图。官方自带的逻辑备份工具是mysqldump。当mysqldump使用参数–single-transaction的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。而由于MVCC的支持，这个过程中数据是可以正常更新的。

对于不支持事务的引擎，使用FTWRL。

### 表级锁

Mysql的表级锁有两种，一种是表锁，一种是元数据锁。

表锁的语法是 lock tables … read/write。与FTWRL类似，可以用unlock tables主动释放锁，也可以在客户端断开的时候自动释放。

另一类表级的锁是MDL（metadata lock)。MDL不需要显式使用，在访问一个表的时候会被自动加上。MDL的作用是，保证读写的正确性。当对一个表做增删改查操作的时候，加MDL读锁；当要对表做结构变更操作的时候，加MDL写锁。

如何安全地给小表加字段？

- 避免MDL锁冲突，可以在alter table语句里面设定等待时间。`ALTER TABLE tbl_name WAIT N add column ... `



## 07讲 行锁功过

### 两阶段锁

两阶段锁协议：在InnoDB事务中，行锁是在需要的时候加上的，但并不是不需要了立即释放，而是等事务结束时才释放。

如果在事务中需要锁多行，应该把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。

### 死锁和死锁检测

出现死锁后，有两种策略：

- 直接进入等待，直到超时。这个超时时间可以通过参数innodb_lock_wait_timeout(默认50s)来设置。
- 发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数innodb_deadlock_detect设置为on，表示开启这个逻辑。

死锁检测是O(n)的操作，因此出现热点数据时，死锁检测要耗费大量的CPU资源。

可以通过修改一行为逻辑上的多行的方式，减少锁冲突。



## 08讲 事务到底是隔离的还是不隔离的














