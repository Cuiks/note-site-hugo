---
title: "Mysql实战45讲笔记"
date: 2021-04-25T09:47:34+08:00
tags: ["mysql"]
draft: true
---

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









