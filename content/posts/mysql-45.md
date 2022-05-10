---
title: "Mysql实战45讲笔记"
date: 2021-07-29T16:46:34+08:00
tags: ["mysql"]
draft: false
---

MySQL实战45讲

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
- 通过执行 `mysql_reset_connection`来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。

#### 查询缓存

- 将参数`query_cache_type`设置成`DEMAND`，这样对于默认的SQL语句都不使用查询缓存。
- 对于确定要使用查询缓存的语句，可以用`SQL_CACHE`显式指定：`select SQL_CACHE * from T where ID=10；`
- MySQL 8.0版本直接将查询缓存的整块功能删掉了

#### 分析器

- 对SQL语句先做词法分析，然后做语法分析。

#### 优化器

- 决定使用哪个索引
- 决定表关联连接顺序

#### 执行器

- 首先判断用户执行权限，然后根据引擎不同调用不同接口获取数据
- 慢查询日志中`rows_examined`字段，表示这个语句扫描了多少行。这个值就是在执行器每次调用引擎获取数据行的时候累加的。



## 02讲 日志系统：一条SQL更新语句是如何执行的

### 日志模块：redo log

- InooDB引擎特有的日志
- WAL技术，全称是`Write-Ahead Logging`，关键点就是先写日志，再写磁盘
- 有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为`crash-safe`。
- `innodb_flush_log_at_trx_commit`这个参数设置成1的时候，表示每次事务的redo log都直接持久化到磁盘。这个参数建议设置成1，这样可以保证MySQL异常重启之后数据不丢失。
- `redo log buffer`是什么？
  - `redo log buffer`是一块内存，用来先存`redo`日志的。在执行`commit`语句的时候，真正把日志写入`redo log`文件

### 重要的日志模块：binlog

- Server层日志
- redo log和binlog区别
  1. redo log是InnoDB引擎特有的；binlog是MySQL的Server层实现的，所有引擎都可以使用。
  2. redo log是物理日志，记录的是“在某个数据页上做了什么修改”；binlog是逻辑日志，记录的是这个语句的原始逻辑，比如“给ID=2这一行的c字段加1 ”。
  3. redo log是循环写的，空间固定会用完；binlog是可以追加写入的。“追加写”是指binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。
- InnoDB引擎执行update操作内部流程 `update T set c=c+1 where ID=2;`
  1. 执行器先找引擎去ID=2的一行。ID是主键，引擎直接用树搜索。如果在内存中则直接返回给执行器，否则需要从磁盘读入内存，然后返回
  2. 执行器拿到引擎给的行数据，加1，得到新一行数据，调用引擎接口，写入新数据
  3. 引擎更新数据到内存，同时记录到redo log，此时redo log处于prepare状态，然后告知执行器执行完成了，随时可以提交事务
  4. 执行器生成binlog，写入磁盘
  5. 执行器调用引擎的提交事务接口，引擎把redo log改成提交（commit）状态，更新完成。
- sync_binlog参数设置成1的时候，表示每次事务的binlog都持久化到磁盘。这个参数也建议设置成1，这样可以保证MySQL异常重启之后binlog不丢失。

### 两阶段提交

- redo log写写入被 拆成了两个步骤：prepare和commit，这就是“两阶段提交”
- 原因
  - 为了让两份日志之间的逻辑一致
- redo log和binlog都可以用于表示事物的提交状态，而两阶段提交就是让这两个状态保持逻辑上的一致



## 03讲 事务隔离：为什么你改了我还看不见

### 隔离性与隔离级别

- ACID（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性）
- 为了解决脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题，于是有了“隔离级别”的概念。

#### 隔离级别

- 读未提交（read uncommitted）：一个事务还没提交时，它做的变更就能被别的事务看到。
- 读提交（read committed）：一个事务提交之后，他做的变更才会被其他事务看到。
- 可重复读（repeatable read)：一个事务执行过程中看到的数据，总是跟这个事务在启动时候看到的数据是一致的。在可重复读隔离级别下，为提交变更对其他事务也是不可见的。
- 串行化（serializable）：对同一行记录，读写都会加锁。当出现锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

#### 隔离级别的实现

- 数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准
- “可重复读”隔离级别下
  - 视图是在事务启动时创建的，整个事务存在期间都用整个视图
- “读提交”隔离级别下
  - 视图在每个SQL语句开始执行的时候创建的
- “读未提交”隔离级别下
  - 没有视图概念
- “串行化”隔离级别下
  - 直接用加锁的方式来避免并行访问

#### 事务隔离的实现

- MySQL中，每条记录在更新的时候都会同时记录一条回滚操作。记录上的最新值，通过回滚操作，都可以得到前一个状态的值
- 查询记录的时候，不同时刻启动的事务会有不同的read-view，这就是数据库的多版本并发控制（MVCC）
- 回滚日志会在不需要的时候才删除。系统判断没有事务在需要时
- 为什么建议尽量不使用长事务
  - 长事务会存在很老的事务视图。占用大量存储空间
  - 长事务还占用锁资源

#### 事务的启动方式

- 显式启动事务语句， begin 或 start transaction。配套的提交语句是commit，回滚语句是rollback。
- set autocommit=0，这个命令会将这个线程的自动提交关掉。意味着如果只执行一个select语句，这个事务就启动了，而且并不会自动提交。这个事务持续存在直到主动执行commit 或 rollback 语句，或者断开连接。
- 建议`set autocommit=1`，通过显式语句启动事务

查找持续时间超过60S的事务：

`select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60`



## 04讲 深入浅出索引（上）

索引的出现就是为了提高查询的效率，类似书的目录一样。

### 索引的常见模型

- 哈希表
  - 哈希表是一种以键-值（key-value）存储数据的结构，我们只要输入待查找的值即key，就可以找到其对应的值即Value。
  - 哈希表这种结构适用于只有等值查询的场景，比如Memcached及其他一些NoSQL引擎。
- 有序数组
  - 有序数组在等值查询和范围查询场景中的性能就都非常优秀
  - 查找复杂度O(log(N))
  - 有序数组索引只适用于静态存储引擎，在需要更新数据的时候比较麻烦
- 二叉搜索树
  - 查找复杂度O(log(N))
  - 更新复杂度O(log(N))
  - N叉树：多个儿子之间大小保证从左到右递增
  - 实际大多数的数据库并不适用二叉树，而是N叉树。因为索引不止存在内存中，还要写到磁盘上
  - 为了让一个查询尽量少的度期盼，必须让查询访问尽量少的数据库，所以使用N叉树

### InnoDB索引模型

- 在InnoDB中，表都是根据主键顺序以索引的形式存放的，这种存储方式的表成为索引组织表。InnoDB使用了B+树索引模型，所以数据都是存储在B+树中。

- 每一个索引在InnoDB中对应一棵B+树。

索引类型分为`主键索引`和`非主键索引`：

- 主键索引的叶子节点存的是整行数据。InnoDB中，主键索引也被称为`聚簇索引`。
- 非主键索引的叶子节点存的是主键的值。InnoDB中，非主键索引也被称为`二级索引`。
- 基于主键索引和普通索引的查询有什么区别？
  - 主键查询方式，则只需要搜索ID这棵B+树；
  - 普通索引查询方式，则需要先搜索k索引树，得到ID的值为500，再到ID索引树搜索一次。这个过程称为回表。
  - 基于非主键索引的查询需要多扫描一棵索引树。因此，我们在应用中应该尽量使用主键查询。

### 索引维护

- 数据挪动、页分裂、页合并 影响性能
- 主键长度越小，普通索引的叶子节点就越小，普通索引占用的空间也就越小
  - 所以自增主键往往更合理
- 重建索引`alter table T engine=InnoDB`



## 05讲 深入浅出索引（下）

- 回到主键索引树搜索的过程，我们称为回表。

### 覆盖索引

- 在查询中，索引k已经“覆盖了”我们的查询需求（索引k记录了主键ID），我们称为`覆盖索引`。
  - select ID from T where k between 3 and 5，这时ID的值已经在k索引树上了
- 由于覆盖索引可以减少树的搜索次数，显著提升查询性能，所以使用覆盖索引是一个常用的性能优化手段。
- 在市民信息表，建立身份证号和名字联合索引
  - 当根据身份证号查询名字是，可以减少回表次数

### 最左前缀原则

在建立联合索引的时候，如何安排索引内的字段顺序。

- 第一原则是，如果通过调整顺序，可以少维护一个索引，那么这个顺序往往就是需要优先考虑采用的。
- 然后考虑空间。

### 索引下推

- 可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。

- `select * from tuser where name like '张%' and age=10 and ismale=1;`，根据索引搜索的时候，InnoDB在(name,age)索引内部就判断了age，对于不符合的记录，直接判断并跳过。只需要对ID4、ID5这两条记录回表取数据判断，就只需要回表2次。

## 06讲 全局锁和表锁

根据加锁的范围，Mysql的锁可以分为全局锁、表级锁、行锁

### 全局锁

- 对整个数据库实例加锁，一般用于全库逻辑备份。

- MySQL提供了一个加全局读锁的方法，命令是 `Flush tables with read lock (FTWRL)`，当你需要让整个库处于只读状态的时候，此时其他线程的数据修改、建表修改表结构、更新类事务的提交都会被阻塞。

- 对于支持事务的引擎，可以通过事务，保证拿到一致性视图。官方自带的逻辑备份工具是mysqldump。当mysqldump使用参数`–single-transaction`的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。而由于MVCC的支持，这个过程中数据是可以正常更新的。

- 对于不支持事务的引擎，使用FTWRL。

### 表级锁

- Mysql的表级锁有两种，一种是表锁，一种是元数据锁。

- 表锁的语法是 lock tables … read/write。与FTWRL类似，可以用unlock tables主动释放锁，也可以在客户端断开的时候自动释放。

- 另一类表级的锁是MDL（metadata lock)。MDL不需要显式使用，在访问一个表的时候会被自动加上。MDL的作用是，保证读写的正确性。当对一个表做增删改查操作的时候，加MDL读锁；当要对表做结构变更操作的时候，加MDL写锁。

如何安全地给小表加字段？

- 首先要解决长事务，防止事务不提交，一直占着MDL锁
- 避免MDL锁冲突，可以在alter table语句里面设定等待时间。`ALTER TABLE tbl_name [WAIT N | NOWAIT] add column ... `



## 07讲 行锁功过

- MySQL的行锁是在引擎层由各个引擎自己实现，不是所有引擎都支持行锁。InnoDB支持，MyISAM不支持

### 两阶段锁

- 两阶段锁协议：在InnoDB事务中，行锁是在需要的时候加上的，但并不是不需要了立即释放，而是等事务结束时才释放。

- 如果在事务中需要锁多行，应该把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。

### 死锁和死锁检测

- 当并发系统中不同线程出现循环资源依赖，设计的线程都在等待别的线程释放资源时，就会导致几个线程都进入无限等待的状态，成为死锁

- 出现死锁后，有两种策略：
  - 直接进入等待，直到超时。这个超时时间可以通过参数`innodb_lock_wait_timeout`(默认50s)来设置。
  - 发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数`innodb_deadlock_detect`设置为on，表示开启这个逻辑。
  - 正常情况下建议第二种侧策略“死锁检测”。

- 死锁检测是O(n)的操作，因此出现热点数据时，死锁检测要耗费大量的CPU资源。
- 如何解决这种热点行更新导致的性能问题：
  - 能够确保业务不会出现死锁，临时关闭死锁检测
  - 控制并发度
    - 在服务端控制并发。对于相同行的更新，在进入引擎之前排队。
  - 可以通过将一行改成逻辑上的多行的方式，减少锁冲突。比如合并多次操作一次进行更新。
    - 业务逻辑变复杂代价。



## 08讲 事务到底是隔离的还是不隔离的

- MySQL里，有两个“视图”的概念

  - 一个是view。它是一个用查询语句定义的虚拟表，在调用的时候执行查询语句并生成结果。创建视图的语法是`create view ...`，而它的查询方法与表一样。
  - 另一个是InooDB在实现MVCC时用到的一致性读视图，即`consistent read view`，用于支持RC(Read Committed,读提交)和RR(Repeatable Read,可重复读)隔离级别的实现。

  没有物理结构，作用是事务执行期间用来定义“我能看到什么数据”。

### “快照”在MVCC里是怎么工作的

- 在可重复读隔离级别下，事务在启动的时候就“拍了个快照”。而且这个快照是基于整库的
- 实现原理
  - InnoDB里面每个事务有一个唯一的事务ID，叫做`transaction id`，事务开始时向InnoDB的事务系统申请，按照申请顺序严格递增
  - 每行数据也是有多个版本的。每次事务更新数据的时候，都会生成一个数据版本，并且把`transaction id`赋值给这个数据版本的事务ID，记为`row trx_id`。同时，旧的数据版本要保留，并且在新的数据版本中，能够有信息可以直接拿到它。
    - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210719144213.png)
    - 上图中三个虚线箭头即`undo log`，V1、V2、V3需要并不是物理上真实存在的，而是每次需要的时候根据当前版本和`undo log`计算出来
  - 因此，一个事务只需要在启动的时候声明：“以我的启动时刻为准”。如果数据版本是在启动之前生成，则认可。否则，不认可，需要找到他的上一个版本（一直前一个版本直到找到启动时刻之前版本）
- 具体实现
  - InnoDB为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在“活跃（启动了但还没提交）”的所有事务ID。
  - 数组中事务ID的最小值记为低水位，已创建过的事务ID的最大值记为高水位
    - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210719145229.png)
  - 这个视图数组和高水位，就组成了当前事务的一致性视图（read-view）
  - 数据版本的可见性规则，就是基于数据的`row trx_id`和这个一致性视图的对比结果得到的。
  - 对于当前事务的启动瞬间来说，一个数据版本的`row trx_id`，有以下几种可能：
    - 落在绿色部分，表示这个版本是已提交的事务或者是当前事务自己生成的，这个数据是可见的。
    - 落在红色部分，表示这个版本是由将来的事务生成，肯定不可见
    - 落在黄色部分：
      - 若`row trx_id`在数组中，表示这个版本是由还没有提交的事务生成的，不可见
      - 若`row trx_id`不在数组中，表示这个版本是已经提交了的事务生成的，可见。
- InnoDB利用了“所有数据都有多个版本”的特性，实现了“秒级创建快照”的能力。
- 一个数据版本，对于一个事务视图来说，除了自己的更新总是可见以外，有三种情况：
  - 版本未提交，不可见
  - 版本已提交，但是是在视图创建后提交的，不可见
  - 版本已提交，而且是在视图创建前提交的，可见



### 更新逻辑

- 更新数据都是先读后写的，而这个读，只能读当前值，称为“当前读”（current read）。



- 可重复读的核心就是一致性读(consistent read)；而事务更新数据时，只能用当前读。如果当前的记录的行锁被其他事务占用，就需要进入锁等待。
- 读提交的逻辑和可重复读逻辑类似，最主要区别：
  - 可重复读隔离级别下，只需要在事务开始的时候创建一致性视图，之后事务里的其他查询都共用这个一致性视图
  - 读提交隔离级别下，每一个语句执行前都会重新算出一个新的视图

### 小结

- InnoDB的行数据有多个版本，每个数据版本都有自己的`row trx_id`，每个事务或者语句都要自己的一致性视图。
- 普通查询语句是一致性读，一致性读会根据`row trx_id`和一致性视图确定数据版本的可见性。
- 
  - 对于可重复读，查询只承认在事务启动之前就已经提交完成的数据
  - 对于读提交，查询只承认在语句启动前就已经提交完成的数据
  - 而当前读，总是读取已经提交完成的最新版本



## 09讲 普通索引和唯一索引

前提：对于业务保证唯一性的字段：

#### 查询过程

- 普通索引，查找到满足条件的第一条记录后，需要查找下一个记录，知道碰到第一个不满足条件的记录。
- 唯一索引，有雨索引定义了唯一性，查找到第一个满足条件的记录后，就会停止继续检索

- 以上两者性能差距**微乎其微**。

#### 更新过程

- change buffer

  - 当需要更新一个数据页时，如果数据页在内存中就直接更新，如果数据页不在内存中，在不影响数据一致性的前提下，InnoDB会将这些更新操作缓存在`change buffer`中，这样就不需要从磁盘中读取这个数据页了。
  - `change buffer`在内存中有拷贝，也会被写入到磁盘上
  - 将`change buffer`中的操作应用到原数据页得到最新结果的过程称为`merge`。
    - 访问数据页会触发merge
    - 系统有后台线程定期merge
    - 数据库正常关闭过程中，也会执行merge操作
  - 将更新操作先记录在`change buffer`，减少读磁盘，语句的执行速度会明显提升。而且，数据读入内存需要占用`buffer pool`，所以这种方式还能后避免占用内存，提高内存利用率。
  - 什么条件下可以使用`change buffer`
    - 唯一索引所有的更新操作都要判断这个操作是否违反唯一性约束。判断过程需要将数据页读入内存才能判断，所以没必要使用`change buffer`
    - 只有普通索引可以使用`change buffer`
  - `change buffer`用的是`buffer poll`的内存。通过参数`innodb_change_buffer_max_size`来动态设置。50表示自多只能占用`buffer poll`的50%。

- InnoDB插入新纪录流程

  1. 要更新的目标页在内存中

     - 唯一索引：找到3和5之间的位置，判断没有冲突，插入这个值

     - 普通索引：找到3和5之间的位置，插入这个值

  2. 要更新的目标也不在内存中

     - 唯一索引：将数据页读入内存，判断没有冲突，插入这个值
     - 普通索引：将更新记录在`change buffer`

     将数据从磁盘读入内存涉及随机IO的访问没事数据库里成本最高的操作之一。`change buffer`因为减少了随机磁盘访问，所以对更新性能的提升是会很明显的。

#### `change buffer`的使用场景

- 因为merge的时候是真正进行数据更新的时刻，而`change buffer`的主要目的是将记录的变更动作缓存下来，所以在一个数据页merge之前，`change buffer`记录的变更越多，收益就越大
- 因此，对于写多读少的业务来说，`change buffer`的使用效果最好。例如账单类、日志类的系统
- 反之，写入之后直接做查询，随机访问IO的次数不会减少，反而增加了`change buffer`的维护代价

#### 索引选择和实践

- 普通索引和唯一索引，查询能力没差别，主要考虑更新性能的影响，建议尽量选择普通索引
- 如果所有更新后面，都马上伴随着对这个记录的查询，应该关闭`change buffer`

#### `change buffer`和`redo log`

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210719161232.png)

- 更新操作如下：
  1. Page1 在内存中，直接更新内存
  2. Page2 没在内存中，就在内存的`change buffer`区域，记录更新信息
  3. 将上述两个动作记录到`redo log`（图中3和4）
- `redo log`主要节省的随机写磁盘的IO消耗（转成顺序写），而`change buffer`主要节省的则是随机读磁盘的IO消耗



## 10讲 MySQL为什么有时候会选错索引

#### 优化器的逻辑

- 优化器的判断标准：扫描行数、是否使用临时表、是否排序
- MySQL在真正开始执行之前，并不能准确地知道满足条件的记录条数，只能根据统计信息来估算记录数。这个统计信息就是索引的“区分度”
- 一个索引上不同值越多，索引的区分度越好。一个索引上不同的值的个数，称之为“基数(cardinality)”。基数越大，索引的区分度越好
- `show index from t`。查看一个索引的基数
- 优化器也会把回表扫描的代价算上

#### MySQL是怎样得到索引基数的：

- 采样统计的方法：
  - 默认选择N个数据页，得到平均值，然后乘以页面数
- 变更的数据行超过`1/M`的时候，会自动触发重新做一次索引统计



- MySQL选错索引根本原因还是没能准确地判断出扫描行数
- `analyze table t`可以重新统计索引信息

#### 索引选择异常和处理

- 一种方法是：采用`force index`强行选择一个索引
- 第二种方法：可以考虑修改语句，引导MySQL使用我们期望的索引（通过修改where和order条件）
- 第三种方法：可以新建一个更合适的索引，来提供给优化器做选择，或删除误用的索引



## 11讲 怎么给字符串字段加索引

- `alter table SUser add index index1(email);` 为email字段添加索引（index1）

- `alter table SUser add index index2(email(6));` 为email字段前6个字节添加索引（index2）

- 使用前缀索引，定义好长度，就可以做到既节省空间，又不用额外增加太多的查询成本

- 如何确定使用多长的前缀：

  - ```sql
    select 
      count(distinct left(email,4)）as L4,
      count(distinct left(email,5)）as L5,
      count(distinct left(email,6)）as L6,
      count(distinct left(email,7)）as L7,
    from SUser;
    ```

#### 前缀索引对覆盖索引的影响

- 前缀索引即使包含所有信息，也会回到id索引再查一下，保证索引查询信息的完整
- 使用前缀索引就用不上覆盖索引对查询性能的优化了

#### 其他方式

- 倒序存储。吧身份证倒序存储，使用前6位做索引，可以保证足够的区分度
- 使用hash字段。添加整数字段保存身份证的校验码，同时添加索引
- 两种方法相同点
  - 都不支持范围查询
- 两种方法相异点
  - 从占用空间来看：倒序存储方式在主键索引上，不会消耗额外的存储空间，hash方式需要增加一个字段。
  - 在CPU消耗方面：倒序方式每次写和读的时候，需要额外调用一次reverse函数，hash方式需要额外调用一次crc32()函数，reverse消耗资源更小
  - 查询效率：hash字段方式更稳定



## 12讲 为什么MySQL会“抖”一下

- 现象：一条SQL语句，正常执行的时候特别快，但是有时不知道怎么回事，就会特别慢，并且很难复现。随机且持续时间很短

#### 你的SQL语句为什么变“慢”了

- 当内存数据页跟磁盘数据页内容不一致的时候，我们称这个内存页为“脏页”。内存数据写入磁盘后，内存和磁盘上的数据页就一致了，称为“干净页”。
  - 不论脏页还是干净页，都在内存中。
- MySQL偶尔“抖”一下的那个瞬间，可能就是在刷脏页（flush）
- 什么情况下会引发数据库的flush过程？
  - `redo log`满了，系统停止所有更新操作。把`checkpoint`往前推进，`redo log`留出空间可以继续写。
  - 系统内存不足。当需要新内存页时，会淘汰一些内存页，如果淘汰的是“脏页”，需要先将脏页写到磁盘。
  - MySQL认为系统“空闲”的时候。
  - MySQL正常关闭的时候。
- InnoDB需要有控制脏页比例的机制，来尽量避免上面的前两种情况

#### Innodb刷脏页的控制策略

- 首先需要正确高速InnoDB所在主机的IO能力，这样InnoDB才能知道需要全力刷脏页的时候，可以刷多快
  - `innodb_io_capacity`参数，会高速InnoDB磁盘能力。建议设置成磁盘的IOPS
- InnoDB刷盘速度主要参考两个因素：
  - 脏页比例
    - `innodb_max_dirty_pages_pct`参数是脏页比例上限，默认为75%
  - `redo log`写盘速度
- `innodb_flush_neighbors`参数控制是否flush相邻的脏页，找"邻居"可能会导致查询更慢。建议设置为0。MySQL8.0默认为0



## 13讲 为什么表数据删掉一半，表大小不变

- InnoDB表含两部分：表结构定义和数据
  - MySQL8.0以前，表结构是存在以`.frm`为后缀的文件里。
  - MySQL8.0以后，允许把表结构定义放在系统数据表中了。因为表结构定义占用的空间很小。

#### 参数`innodb_file_per_table`

- 表数据既可以存在共享表空间里，也可以是单独的文件。由参数`innodb_file_per_table`控制。
  - OFF。表的数据放在共享表空间，也就是跟数据字典放在一起
  - ON。每个InnoDB表数据存储在一个以`.ibd`为后缀的文件中
  - MySQL5.6.6开始 默认为ON

#### 数据删除流程

- 用B+树删除一条数据的时候，只会吧记录标记为删除。如果之后再插入的时候，会复用这个位置。但是磁盘文件的大小并不会缩小
- 如果删掉一个数据页上的所有记录，整个数据页会被标记位可复用
  - 如果相邻的两个数据页利用率都很小，系统就会把这两个页上的数据合到其中一个页上，另外一个数据页就被标记为可复用。
  - 用`delete`删除整个表的数据，所有的数据页都会被标记位可复用，但是在磁盘上，文件不会变小
- 数据页的复用跟记录的复用是不同的。
  - 记录的复用，只限于符合范围条件的数据。
  - 而当整个页从B+树里面摘掉以后，可以复用到任何位置。
- 经过大量增删改的表，都可能存在大量的“空洞”。

#### 重建表

- 方法1：新建相同表结构表B，然后按照主键ID递增顺序，把表A的数据插入到表B

- 方法2：`alter table A engine=InnoDB`。MySQL会自动完成转存数据、交换表名、删除旧表的操作。该过程 DDL 不是 Online 的

  - 5.6版本后引入 Online DLL ：

    1. 建立临时文件，扫描A主键的所有数据页
    2. 用数据页中表A 的记录生成B+树，存储到临时文件中
    3. 生成临时文件的过程中，所有对A的操作记录在一个日志文件（`row log`）中
    4. 临时文件生成后，将日志文件中的操作应用到临时文件，得到一个逻辑数据上与表A相同的数据文件
    5. 用临时文件替换表A的数据文件

    这个方案在重建表的过程中允许对表 A 做增删改操作，也就是 Online DDL 名字的来源。
    
  - `alter`语句在启动的时候需要获取 MDL 写锁，但是这个写锁在真正拷贝数据的之前就退化成了读锁。MDL读锁不会阻塞增删改操作

#### Online 和 inplace

- `alter table t engine=innodb,ALGORITHM=inplace;`
  - 整个DDL过程在 InnoDB 内部完成。对于 server 层来说是一个“原地”操作。
- `alter table t engine=innodb,ALGORITHM=copy;`
  - `copy`表示强制拷贝表。会在 server 层创建临时表
- 逻辑关系
  1. DDL 过程如果是 Online 的，就一定是 inplace 的
  2. 反过来未必，也就是说 inplace 的DDL，有可能不是 Online 的。

`optimize table`、`analyze table` 和 `alter table`这三种方式重建表的区别：

- MySQL5.6开始，`alter table t engin=InnoDB`默认就是使用`inplace`方式
- `analyze table t`其实不是重建表，只是对表的索引信息做重新统计，没有修改数据，这个过程加了 MDL 读锁
- `optimize table t`等于上面两个都做



## 14讲 count(*)这么慢，我该怎么办

#### count(*) 的实现方式

- 在不同的MySQL引擎中，`count(*)`有不同的实现方式
  - MyISAM引擎把一个表的总行数存在了磁盘上（前提是没有where条件）
  - InnoDB引擎：在执行`count(*)`的时候，需要把数据一行一行的从引擎里读出来，然后累积计数

- 为什么InnoDB不把行数存起来？
  - 因为即使在同一个时刻的多个查询，由于多版本并发控制（MVCC）的原因，InnoDB表“应该返回多少行”也是不确定的。
  - MySQL优化器会找到最小的树（普通索引树<主键索引树）来遍历。**在保证逻辑正确的前提下，尽量减少扫描的数据量，是数据库系统设计的通用法则之一**。
- `show table status`命令的`TABLE_ROWS`显示的行数是估算结果，非常不准确。
- 小结
  - MyISAM表`count(*)`很快，但是不支持事务
  - `show table status`返回很快，但是不准确
  - InnoDB表直接`count(*)`会遍历全表，结果准确，但是会导致性能问题。

#### 用缓存系统保存计数

- 用 Redis服务来保存表的行数，插入加1，删除减1
  - 读和更新操作都很快，缓存系统很可能会丢失更新
  - Redis异常重启会导致数据不准确（可以通过重启后计算总行数更新来解决）
- 根本问题：
  - 不同事务查到的结果不同，无法使用Redis记录

#### 在数据库保存计数

- MySQL单独一张表保存行数
- 问题：同样不能解决不同事务结果不同问题



#### 不同的count用法

- `count(*)`、`count(主键id)`、`count(字段)`、`count(1)`
- 分析性能差别原则：
  - server层要什么就给什么
  - InnoDB只给必要的值
  - 现在的优化器只优化了`count(*)`的语义为“取行数”，其他“显而易见”的优化并没有做
- `count(主键id)`
  - InnoDB引擎遍历整张表，把每一行的主键id值取出来，返回给sever层。server层拿到后，判断不可能为空的，就按行累加
- `count(1)`
  - InnoDB引擎遍历整张表，但不取值。server层对于返回的每一行，放一个数字“1”进去，判断是不可能为空的，按行累加。
  - 比`count(主键id)`快。因为从引擎返回id会涉及到解析数据行以及拷贝字段值的操作
- `count(字段)`
  - 如果这个“字段”是定义`not null`，一行行的从记录读取这个字段，判断不能为`null`，按行累加
  - 如果这个“字段”是定义允许`null`，那么执行的时候，判断有可能是`null`，还要把值取出来再判断一下，不是`null`才累加
- `count(*)`
  - 并不会把全部字段取出来，而是专门做了优化，不取值。`count(*)`肯定不是`null`，按行累加
- 效率排序
  - `count(字段)` < `count(主键id)`<`count(1)`≈`count(*)`
  - 建议尽量使用`count(*)`



## 15讲 日志和索引相关问题

- 正常运行的实例，数据写入后的最终落盘，是从`redo log`更新过来时还是从`buffer pool`
  - `redo log`并没有记录数据页的完整数据，所以他没有能力自己去更新磁盘数据页
  - 1、正常运行的实例，最终数据落盘，就是把内存的脏页写盘，这个过程跟`redo log`毫无关系。
  - 2、在崩溃场景中，InnoDB如果判断到一个数据页可能在崩溃恢复的时候丢失了更新，就会将它读入内存，然后让`redo log`更新内存内容。更新完成后，内存页变成脏页，就回到了上面情况的状态。



## 16讲 "orderby"是怎样工作的

#### 全字段排序

- MySQL会给每一个线程分配一块内存用于排序，称为`sort_buffer`
- `sort_buffer_size`参数。如果要排序的数据量小于该值则排序在内存中完成，否则需要使用磁盘文件辅助排序。

#### rowid排序

- 如果MySQL认为排序的单行长度太大会怎么做？
- `max_length_for_sort_data` 如果单行长度超过这个值，MySQL就认为单行太大。
- 此时，只把要排序的列和主键id放入`sort_buffer，排序后再按照id的值从原表取出需要字段。称为`rowid`排序
- `rowid`
  - 有主键的InnoDB，rowid就是主键ID
  - 没有主键的InnoDB，rowid由系统生成的
  - MEMORY引擎不是索引组织表，可以看做数组。rowid就是数组下标

#### 全字段排序 VS rowid排序

- 如果内存够买就多利用内存，尽量减少磁盘访问
- 通过创建`where`条件、`order`条件、结果字段的联合索引可以提交order效率。“覆盖索引”
  - select city,name,age from t where city='杭州' order by name limit 1000  ;
  - `alter table t add index city_user_age(city, name, age);`



## 17讲 如何正确地显示随机消息

#### 内存临时表

- `select word from words order by rand() limit 3; `
  - 使用临时表；需要执行排序操作。
- `order by rand()`使用了内存临时表，内存临时表排序的时候，使用了rowid排序方法

#### 磁盘临时表

- `tmp_table_size`限制了内存临时表的大小，默认16m。如果临时表大小超过，就会转成磁盘临时表
- 磁盘临时表使用引擎默认为InnoDB。
- 优先队列排序算法

#### 随机排序方法

1. 取得整个表的行数C
2. 取得`Y = floor(C * rand())`。`floor`函数表示取整数部分
3. 用`limit Y,1`取一行

## 18讲 为什么SQL逻辑相同，性能差异却巨大

#### 案例一：条件字段函数操作

- 对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能

#### 案例二：隐式类型转换

- `select * from tradelog where tradeid=110717;`
  - `tradeid`字段有索引，但是这条语句扫描了全表
  - 原因：`tradeid`是`varchar(32)`，而输入的参数却是整型，所以需要做类型转换
- MySQL中，字符串和数字作比较的话，是将字符串转换成数字

#### 案例三：隐式字符编码转换

- 如果两个类型的字符串在做比较的时候，先把`utf8`转成`utf8mb4`，在做比较
- 所以，如果两个字符类型不同的字段作比较的时候，需要先使用`CONVERT()`函数转换编码，导致优化器放弃走索引

- 解决方案
  - 修改字符集为`utf8mb4`
  - 强制转换驱动表，避免被驱动表的字符集转换。



## 19讲 为什么只查一行的语句，也执行这么慢

#### 第一类：查询长时间不返回

- `show processlist`

##### 等MDL锁

- `Waiting for table metadata lock`
- 表明现在有一个线程正在表t上请求或者持有MDL写锁
- 处理方式
  - kill掉持有MDL写锁的进程

##### 等flush

- `Waiting for table flush`

##### 等行锁

- 查询是占着这个写锁
  - `select * from t sys.innodb_lock_waits where locked_table=`'test'.'t'`\G`
    - `blocking_pid`

### 第二类：查询慢

- 字段没有索引，扫描函数多
- 可重复读隔离级别下，视图过长，需要回滚多次
  - 例：`sessionA` 启动事务；`sessionB`更新了100万次；`sessionA`查询；此时`sessionA`需要回滚100万次的回滚才能拿到值。



## 20讲 幻读是什么

#### 幻读是什么

- 幻读指的是一个事务在前后两次查询同一个范围的时候，后一次查询看到了前一次查询没有看到的行。
- 说明
  - 在可重复读隔离级别下，普通的查询是快照读，是不会看到别的事务插入的数据的。因此，幻读在“当前读”下才会出现
  - 幻读专指“新插入的行”

#### 幻读有什么问题？

- 语义上的破坏
- 数据一致性的问题
- 即使把所有记录都加上锁，还是阻止不了新插入的记录

#### 如何解决幻读

- 间隙锁（`gap lock`）
- `select * from t where d=5 for update`，不止给数据库已有的6个记录加上了行锁，还同时加了7个间隙锁。这样就确保了无法再插入新纪录
- 跟间隙锁存在冲突关系的，是“往这个间隙中插入一个记录”这个操作
- 间隙锁和行锁合称为`next-key lock`
- 间隙锁是在可重复读隔离级别下才会生效的，所以把隔离级别设置为读提交的话，就没有间隙锁了。但同时，要解决可能出现的数据和日志不一致的问题，需要把`binlog`格式设置为`row`。

间隙锁和`next-key lock`的引入，帮我们解决了幻读问题，但同时也带来了一些困扰

- 间隙锁的引入，可能会导致同样的语句锁住更大的范围，这其实是影响并发度的



## 21讲 为什么只改一行，锁这么多

- 本篇默认可重复读隔离级别

#### 两个“原则”、两个“优化”、一个“bug”

- 原则1：加锁的基本单位是`next-key lock`。`next-key lock`是前开后闭区间
- 原则2：查找过程中访问到的对象才会加锁
- 优化1：索引上的等值查询，给唯一索引加锁的时候，`next-key lock`退化为行锁
- 优化2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，`next-key lock`退化为间隙锁。
- 一个bug：唯一索引上的范围查询会访问到不满足条件的第一个值为止



## 22讲 MySQL"饮鸩止渴"提高性能的方法

#### 短连接风暴

- 短连接导致业务高峰期出现的连接数突然暴涨的情况
- `max_connections`控制MySQL实例同事存在的连接数的上限

##### 方法1：先处理掉哪些占着链接但是不工作的线程

- `show processlist`为`sleep`的线程
- 可能会误杀事务。
  - 通过查`information_schema`库的`innodb_trx`表判断是否事务中

##### 方法2：减少链接过程的损耗

- 使用`–skip-grant-tables`参数启动。整个MySQL会跳过所有的权限验证
  - 饮鸩止渴。风险极高

#### 慢查询性能问题

- 索引没有设计好
- 语句没写好
- MySQL选错了索引
  - 可以通过`force index`强制使用某个索引

#### QPS突增问题



## 23讲 MySQL怎么保证数据不丢的

- 只要redo log和binlog保证持久化到磁盘，就能确保MySQL异常重启后，数据可以恢复。

#### binlog的写入机制

- binglog写入逻辑：事务执行过程中，先把日志写到`binlog cache`，事务提交的时候，再把`binlog cache`写到binlog文件中，事务提交的时候，执行器把`binlog cache`里的完整事务写入到binlog中，并清空`binlog cache`
- 一个事务的binlog是不能被拆开的。
- 通过参数控制写过`binlog files`和写入磁盘的时机

#### redo log的写入机制

- `redo log`先写到`redo log buffer`
- 通过参数控制`redo log buffer`写入`page cache`和写入磁盘的时机
- InnoDB后台线程，每隔1秒。就会把`redo log buffer`中的日志调用`write`写入文件系统`page cache`，然后调用`fsync`持久化到磁盘
- `redo log buffer`占用的空间达到`innodb_log_buffer_size`一半的时候，后台线程会主动写盘
- 并行的事务提交的时候，顺带将这个事务的`redo log buffer`持久化到磁盘

#### 如果MySQL出现了IO性能瓶颈，可以通过哪些方法来提升性能？

- 设置`binlog_group_commit_sync_delay`和`binlog_group_commit_sync_no_delay_count`参数，通过延迟调用`fsync`，减少binlog写盘次数。
  - 可能会增加语句的响应时间，但没有丢失数据的风险
- 设置`sync_binlog`大于1。每次提交事务都会`write`，累计N个事务后才`fsync`
  - 肯主机掉电可能会丢失binlog日志
- 设置`innodb_flush_log_at_trx_commit`为2。控制事务提交`redo log`被写入`page cache`
  - 主机掉电会丢失数据

## 24讲 MySQL是怎么保证主备一致的

#### MySQL主备的基本原理

1. 主库接收到客户端更新请求后，执行内部事务的更新逻辑，同事写binlog

2. 备库B跟主库A之间维持一个长连接。主库A内部有一个线程，专门用于服务备库B的这个长连接。

   1. 备库B执行`change master`
   2. 备库B执行`start slave`，启动两个线程。其中一个(`io_thread`)用于与主库建立连接
   3. 主库A检验成功后，从本地读取binlog发给B
   4. 备库B拿到binlog后，写到本地文件，成为中转日志（`relay log`）
   5. 另一个线程(`sql_thread`)读取中转日志，解析出日志里的命令，并执行

   后来多线程复制方案，`sql_thread`会演化成多个线程

#### binlog三种格式对比

- `statement`和`row`  (`mixed`)
- `statement`格式，binlog里面记录的就是SQL语句的原文
  - 存在主备不一致的风险
- `row`格式，binlog里没有了SQL的原文，而是替换成了两个event（删除操作）：`Table_map`和`Delete_rows`
  - `Table_map`：用于说明接下来要操作的表
  - `Delete_rows`：用来定义删除的行为
  - `row`格式缺点是很占空间
  - `row`格式恢复数据更方便
- `mixed`格式，MySQL判断SQL语句是否可能引起主备不一致，如果可能，就用`row`格式，否则就用`statement`

#### 循环复制问题

- 双M结构。节点A和B互为主备关系
- 如何防止双M结构binglog循环复制：
  1. 规定两个库的`server id`必须不同
  2. MySQL在binlog中记录这个命令第一次执行的`server id`。一个备库接到binlog并在重放的过程中，生成与原binglog的`server id`相同的新的`binlog`
  3. 每个库收到主库发来的日志后，先判断`server id`，如果相同，表示是自己生成的，直接丢弃



## 25讲 MySQL是怎么保证高可用的

#### 主备延迟

- 备库执行`show slave status`。返回显示的`seconds_behind_master`表示备库延迟多少秒

#### 主备延迟的来源

- 备库所在机器性能比主库所在机器性能差
- 备库压力大（查询压力等）
- 大事务--必须等主库把事务执行完成才会写入binlog
  - 一次性用delete语句删除太多数据
  - 大表DDL

#### 可靠性优先策略

- 通过设置主库A和备库B均为只读。等待数据同步完成后进行主备切换

#### 可用性优先策略

- 不等主备数据同步，直接主备切换

一般优先使用可靠性优先策略



## 26讲 备库为什么会延迟好几个小时

- 如果备库执行日志的速度持续低于主库生成日志的速度，那这个延迟就有可能成了小时级别。而且对于一个压力持续比较高的主库来说，备库很可能永远都追不上主库的节奏。

#### 备库并行复制能力

- 备库`sql_thread`更新数据，如果使用单线程，就会导致备库应用日志不够快，造成主备延迟（官方的5.6版本之前，MySQL只支持单线程复制）
- 多线程模型：
  - `sql_thread`演变为`coordinator`，负责中转日志和分发事务。真正跟新日志的，变成worker线程
- `coordinator`在分发的时候，需要满足：
  - 不能造成更新覆盖。要求更新同一行的两个事务，必须被分发到同一个worker中
  - 同一个事务不能被拆开，必须放在同一个worker中

#### MySQL5.5版本的并行复制策略

##### 按表分发策略

- 如果两个事务更新不同的表，它们就可以并行
- 如果有跨表的事务，还是要把两张表放在一起考虑

##### 按行分发策略

- 如果两个事务没有更新相同的行，他们在备库上可以并行。该模式要求binlog格式为row

按行并行分发策略在决定线程分发的时候，需要消耗更多的计算资源

约束条件：

- 能够在binlog解析出表名、主键值、唯一索引的值。也就是说必须是row格式
- 表必须有主键
- 不能有外键。级联更新的行不会记录在binlog，这样冲突检测就不准确了

#### MySQL5.6版本的并行复制策略

- 官方5.6版本支持了并行复制，只是支持的粒度是按库并行
  - 用于决定分发策略的hash表里，key就是数据库名
- 相比按表和按行的优势：
  - 构造hash值很快，只需要库名。一个实例上DB不会很多
  - 不要求binlog格式

#### MariaDB的并行复制策略

- 前提：
  - 能够在同一组提交的事务，一定不会修改同一行
  - 主库上可以并行执行的事务，备库上也一定是可以并行执行的
- 实现：
  - 一组一起提交的事务有一个commit_id
  - commit_id写入binlog
  - 备库把相同commit_id的事务分发到多个worker执行
  - 这一组全部执行完成后，coordinator再去取下一批
- 缺点：
  - 事务互相等待。系统吞吐量不够
  - 容易被大事务拖后腿

#### MySQL5.7的并行复制策略

- 优化了MariaDB策略：
  - 同时处于prepare状态的事务，在备库执行时可以并行
  - 处于prepare状态的事务，与处于commit状态的事务之间，在备库执行也是可以并行的

#### MySQL 5.7.22的并行复制策略

- 基于WRITESET的并行复制
- 参数`binlog-transaction-dependency-tracking`
  - COMMIT_ORDER：表示同时处于prepare状态的事务，在备库执行时可以并行
  - WRITESET：对于事务涉及更新的每一行，计算hash值，组成writeset。如果两个事务没有操作相同的行，也就是他们的writeset没有交集，就可以并行
    - hash值是通过“库名+表名+索引名+值”计算出来的
  - WRITESET_SESSION：在WRITESET的基础上多了一个约束，主库同一线程先后执行的两个事务，在备库执行的时候，要保证相同的先后顺序
- 优势：
  - writeset是在主库生成后直接写入到binlog里面的，这样在备库执行的时候，不需要解析binlog内容（event里的行数据），节省了很多计算量
  - 不需要把整个事务的binlog都扫一遍才能决定分发到哪个worker，更省内存；
  - 由于备库的分发策略不依赖于binlog内容，所以binlog是statement格式也是可以的。
- 缺点：
  - 对于“表上没主键”和“外键约束”的场景，WRITESET策略也是没法并行的，也会暂时退化为单线程模型。

## 27讲 主库出问题了，从库怎么办

- 一主多从
- 主库A故障，主库A' 成为新主库，所有从库连接新主库A'

#### 基于位点的主备切换

- 总是找一个“稍微往前”的位点（主库同步的位点往前），然后再通过判断跳过那些在从库B已经执行过的事务

#### GTID（`Global Transaction Identifier`全局事务ID）

- 一个事务在提交的时候生成的，是这个事务的唯一表示
- `GTID = sever_uuid:gno` 两部分组成
  - sever_uuid：实例第一次启动自动生成的
  - gno：整数，初始值为1，每次提交事务加1

#### 基于GTID的主备切换

- 实例B执行`start slave`命令，取`binlog`的逻辑
  1. 实例B指定主库A'，基于主备协议建立连接
  2. 实例B把set_b发给主库A'
  3. 实例A'计算自己存的set_a和set_b的差集
     1. 如果不包含，表示A'已经把实例B需要的binlog删掉了，返回错误
     2. 如果确认全部包含，A'从自己的binlog文件找出第一个不存在set_b的事务，发给B
  4. 之后就从这个事务开始，往后读文件，按顺序取binlog发给B去执行

#### GTID和在线DDL



## 28讲 读写分离有哪些坑

- 主从存在延迟，读从库可能更新不及时

#### 强制走主库方案

- 对于必须拿到最新结果的请求，强制将其发到主库上
- 对于可以读旧数据的请求，才将其发送到从库上

#### Sleep方案

- 主库更新后，读从库之前先sleep一下

#### 判断主备无延迟方案

- 第一种确保主备无延迟方法：每次从库查询前，先判断`seconds_behind_master`是否等于0
- 第二种方法：对比位点确保主备无延迟。
  - Master_Log_File和Read_Master_Log_Pos，表示的是读到的主库的最新位点；
  - Relay_Master_Log_File和Exec_Master_Log_Pos，表示的是备库执行的最新位点。
  - 对比上面的值
- 第三种方法：对比GTID集合确保主备无延迟
  - Auto_Position=1 ，表示这对主备关系使用了GTID协议。
  - Retrieved_Gtid_Set，是备库收到的所有日志的GTID集合；
  - Executed_Gtid_Set，是备库所有已经执行完成的GTID集合。

#### 配合semi-sync（半同步复制）

- 实现：
  1. 事务提交的时候，主库把binlog发给从库
  2. 从库收到binlog后，发回给主库一个ack，表示收到了
  3. 主库收到ack，才给客户端返回“事务完成”的确定
- semi-sync配合判断主备无延迟的方案的问题
  - 一主多从，某些从库执行查询请求会存在过期读现象
  - 持续延迟情况下，可能出现过度等待的问题

#### 等主库位点方案

- `select master_pos_wait(file, pos[, timeout]);`
- 这个命令正常返回的结果是一个正整数M，表示从命令开始执行，到应用完file和pos表示的binlog位置，执行了多少事务。
- 返回值>=0，从库执行，否则去主库执行

#### GTID方案

- ` select wait_for_executed_gtid_set(gtid_set, 1);`
- 逻辑
  - 等待，直到这个库执行的事务中包含传入的`gtid_set`，返回0
  - 超时返回1
- 返回值是0从库执行，否则到主库执行



## 29讲 如何判断一个数据库是不是出问题了

- 主备切换两种场景：主动切换、被动切换
- 被动切换，往往是因为主库出问题了，由HA系统引起

怎么判断主库出问题了？

#### select 1判断

- `select 1`成功返回只能说明这个库的进程还在，并不能说明主库没问题
  - `innodb_thread_concurrency`参数控制InnoDB的并发线程上限，到达上线后不能操作，但是不影响`select 1`
- 并发连接和并发查询
  - `show processlist`展示的是连接
- 在线程进入锁等待以后，并发线程的计数会减一

#### 查表判断

- 创建一个表`health_check`，只放一行数据，然后定期执行查询
  - `select * from mysql.health_check; `
- 问题
  - binlog所在磁盘的空间占用100%时检测不到

#### 更新判断

- `update mysql.health_check set t_modified=now();`
  - 使用一个`timestamp`字段，表示最后一次执行检测的时间
- 问题
  - 主库A和备库B设计为双M结构。如果两个库都用相同的更新命令，可能会出现行冲突，导致主备同步停止
- 解决
  - `mysql.health_check`表添加`server_id`字段
  - `insert into mysql.health_check(id, t_modified) values (@@server_id, now()) on duplicate key update t_modified=now();`
- 上面所有方法都是基于外部检测的。外部检测天然有一个问题，就是随机性

#### 内部统计

- `select * from performance_schema.file_summary_by_event_name where event_name='wait/io/file/innodb/innodb_log_file'\G`
  - 表示统计`redo log`的写入时间
  - `EVENT_NAME`表示统计的类型
- `select * from performance_schema.file_summary_by_event_name where event_name='wait/io/file/sql/binlog'\G`
  - 统计`binlog`
- `update setup_instruments set ENABLED='YES', Timed='YES' where name like '%wait/io/file/innodb/innodb_log_file%';`
  - 打开`redo log`的时间监控
- `select event_name,MAX_TIMER_WAIT  FROM performance_schema.file_summary_by_event_name where event_name in ('wait/io/file/innodb/innodb_log_file','wait/io/file/sql/binlog') and MAX_TIMER_WAIT>200*1000000000;`
  - 检测语句（判断等待时间单次IO是否超过200毫秒）
- `truncate table performance_schema.file_summary_by_event_name;`
  - 出现问题后，清空，继续监控

## 30讲 用动态的观点看加锁

#### 不等号条件里的等值查询

- 在执行过程中，通过树搜索的方式定位记录的时候，用的是“等值查询”的方法。

#### 等值查询的过程

## 31讲 误删数据后怎么办

- 误删数据情况
  - 使用`delete`误删数据行
  - 使用`drop table`或`truncate table`语句误删数据表
  - 使用`drop database`语句误删数据库
  - 使用`rm`命令误删整个MySQL实例

#### 误删行

- 通过Flashback工具通过闪回把数据恢复
  - 原理：修改binlog内容，拿回原库重放
  - 前提：`binlog_format=row`、`binlog_row_image=FULL`
  - 具体实现
    - `insert`语句，binlog event是`Write_rows event`，修改为`Delete_rows event`即可
    - `delete`语句，将`Delete_rows event`修改为`Write_rows event`
    - `Update_rows`，binlog记录数据行修改前后的值，对调这两行的位置即可
- 恢复数据比较安全的做法：恢复出一个备份，或者找一个从库作为临时库，在临时库上执行这些操作，然后再将确认过的临时库的数据，恢复回主库
- 建议
  - `sql_safe_updates`设为on，如果`delete`或者`update`的语句中没有写`where`条件，或者`where`条件里面没有包含索引字段的话，会报错
  - 代码上线前，必须经过SQL审计
- 使用delete命令删除的数据可以通过Flashback恢复，而使用`truncate /drop table`和`drop database`命令删除的数据，就没办法通过Flashback来恢复了
  - 因为这三个命令，记录的binlog是statement格式

#### 误删库/表

- 使用全量备份+增量日志的方式。要求有定期的全量备份并且实时备份binlog
- 恢复流程
  - 取最近依次全量备份
  - 用备份恢复出临时库
  - 从日志备份里，取出误删操作之后的日志
  - 把这些日志，除了误删语句外，全部应用到临时库
- 注意
  - 使用`mysqlbinlog`命令时，加上一个`–database`参数，用来指定误删表所在的库
  - 应用日志要跳过误操作语句
    - 没有GTID模式，只能使用`–stop-position`和`–start-position`参数跳过误操作点
    - 有GTID模式，`set gtid_next=gtid1;begin;commit;`把误操作gtid添加到临时实例的GTID集合
- 加速方法
  - 在`start slave`之前，`change replication filter replicate_do_table = (tbl_name) `，可以让临时库只同步误操作的表

#### 延迟复制备库

- 搭建延迟复制的备库（MySQL5.6引入）
- `CHANGE MASTER TO MASTER_DELAY = N`指定备库持续保持跟主库N秒的延迟

#### 预防误删库/表的方法

- 账号分离。
  - 业务同学只开`DML`权限，不给`truncate/drop`权限
  - DBA团队，日常也使用只读账户，必要时候才使用有更新权限的账户
- 指定操作规范，避免写错要删除的表名
  - 删表之前，先对表进行改名操作，观察一段时间，确保对业务无影响
  - 该表明要求给表明加固定的后缀，然后删表由管理系统执行

#### rm删除数据

- 一个高可用机制的MySQL集群来说，只删除其中一个节点的数据，HA系统就会开始工作，选出一个新的主库，从而保证整个集群的正常工作
- 然后把这个节点的数据恢复回来，再接入整个集群



## 32讲 kill不掉的语句

- MySQL两个kill命令
  - `kill query + 线程id`：表示终止这个线程中正在执行的语句
  - `kill connection + 线程id`：`connection`可缺省，表示断开这个线程的连接，如果这个线程有语句正在执行，也要先停止正在执行的语句

#### 收到kill后，线程做什么

- `kill query thread_id`时：

  1. 把session把运行状态改成`THD::KILL_QUERY`（将变量killed赋值为`THD::KILL_QUERY`）

  2. 给session的执行线程发一个信号

     

#### 为什么执行`kill query`是，不能退出？

- 该线程状态已经被设置为`KILL_QUERY`，但是等待进入InnoDB的循环过程中，并没有去判断线程的状态，因此根本不会进入终止逻辑
- `kill connection`命令时：
  1. 把线程状态设置为`KILL_CONNECTION`
  2. 关闭该线程的网络连接。
- 为什么`show processlist`结果显示为`killed`？
  - 如果一个线程的状态是`KILL_CONNECTION`，就把Command列显示成killed
- kill无效小结
  - kill无效的第一类：线程没有执行到判断线程状态的逻辑
  - 另一类：终止逻辑耗时较长
- 通过客户端`Ctrl + C`命令，也不可以终止线程。客户端只能操作客户端的线程，和服务端通过网络交互，是不可能直接操作服务端线程的

#### 两个对客户端的误解

- 库里表多，连接会慢
  - 不是连接慢，是客户端慢
  - 通过连接命令添加`-A`关掉自动补全，客户端就可以快速返回了（`-q[-quick]`）也可以跳过这个过程
- `-quick`参数不会让服务端加速，相反可能会降低服务端的性能
  - MySQL客户端默认本地缓存`mysql_store_result`。
  - 如果加上`-quick`参数，客户端本地会不使用缓存`mysql_use_result`。采用不缓存方式，如果本地处理很慢，就会导致服务端发送结果被阻塞，因此让服务端变慢。
  - 该参数是让客户端变得更快

## 33讲 查询数据多，会不会把数据库内存打爆

#### 全表扫描对server层的影响

- 引擎查询到的每一行直接放在结果集里面，然后返回给客户端
- 服务端不需要保存完整的结果集。取数流程：
  - 获取一行写入`net_buffer`。`net_buffer_length`参数，默认16k
  - 重复获取行，直到`net_buffer`写满，调用网络接口发送出去
  - 发送成功，清空`net_buffer`，然后继续获取
  - 发送函数返回`EAGAIN`或`WSAEWOULDBLOCK`，表示本地网络栈（`socket send buffer`）写满了，进入等待。知道网络栈重新可写，继续发送
- 一个查询，占用的MySQL内存最大就是`net_buffer_length`
- `socket send buffer`也有限制（默认定义`/proc/sys/net/core/wmem_default`）
- MySQL是“边读边发”的
- `show processlist`状态为`Sending to client`，表示服务器端的网络栈写满了

#### 全表扫描对InnoDB的影响

- `Buffer Poll`会加速查询（如果数据页在内存，直接查询数据页）
  - 内存命中率 `show engine innodb status`中`Buffer pool hit rate`。线上稳定系统保证内存命中率99%以上
- InnoDB内存管理使用LRU算法。通过链表实现
- 链表分为两部分，分成young区域和old区域。新插入的页面放在old区域，满足条件在放到young区域
- 全表扫描，数据页一直被更新到old区域，不会更新到young区域，不影响正常业务



## 34讲 可不可以用join

#### Index Nested-Loop Join (NLJ)

- 先遍历表1，根据表1中取出的值去表2查询满足条件的记录。类似嵌套查询，并且可以用上被驱动表的索引
- 使用join语句，性能比强行拆成多个单表执行SQL语句的性能更好
- 使用join语句，需要让小表做驱动表

#### Simple Nested-Loop Join

- 被驱动表没有用上索引
- 直接全量扫描被驱动表

#### Block Nested-Loop Join (BLJ)

- 被驱动表没有索引
- 把表1数据放入内存`join_buffer`，扫描表2，跟`join_buffer`作对比
- 扫描函数m+n，判断次数同上一个方法m*n，但是是内存操作，速度快很多，性能更好
- 如果内存放不下表1的所有数据，就分段放。

能不能使用join语句？

- 如果可以使用`Index Nested-Loop Join`算法，也就是可以用上被驱动表的索引，是可以的
- 否则扫描次数特别多，尽量不要用

用哪个表做驱动表？

- 总是用小表做驱动表
- 在决定那个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成后，计算参与join的个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表

##### 额外：

- 使用left join，左边的表不一定是驱动表
- 如果需要left join的语义，就不能把被驱动表的字段放在where条件里面做等值判断或不等值判断，必须都写在on里面

## 35讲 join语句怎么优化

#### Multi-Range Read优化 （MRR）

- 主要目的是尽量使用顺序读盘
- 因为大多数的数据都是按照主键递增顺序插入得到的，所以我们可以认为，如果按照主键的递增顺序查询的话，对磁盘的读比较接近顺序读，能够提升读性能
- `set optimizer_switch="mrr_cost_based=off"`
- MRR会把查询到的主键保存在`read_rnd_buffer`，然后排序，加快回表查询速度
- MRR能提升性能的核心在于该查询是范围查询

#### Batch Key Access （BKA）

- 对NLJ算法的优化
- 把表1的数据取出一部分，放在临时内存`join_buffer`，然后一起去表2查询
- `set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';`

#### BNL算法的性能问题

- 被驱动表是一个大的冷数据表，冷表数据量会进入`Buffer Pool`的yound区域，并且可能无法淘汰，影响正常业务。
- 解决
  - 考虑增大`join_buffer_size`的值，减少对被驱动表的扫描次数

#### BNL转BKA

- 方法1：直接在被驱动表上建索引，直接转成BKA算法
- 方法2：考虑使用临时表，临时存储需要的数据。给临时表添加索引
  - `create temporary table temp_t(id int primary key, a int, b int, index(b))engine=innodb;`

#### 扩展-hash join

- 如果`join_buffer`维护的不是一个无序数组，而是一个哈希表，整条语句执行速度就快多了

## 36讲 为什么临时表可以重名

- 内存表：指的是使用Memory引擎，这种表数据都保存在内存里，系统重启会被清空，但是表结构还在
- 临时表：可以使用各种引擎。根据引擎不同数据写在磁盘或者内存

#### 临时表的特性

- 建表语句：`create temporary table …`
- 临时表只能被创建它的session访问，对其他线程不可见
- 临时表可以与普通标同名
- 同一session内有同名的临时表和普通标的时候，`show create`语句，以及增删改查语句访问的是临时表
- `show tables`不显示临时表

#### 临时表的应用

- 分库分表系统的跨库查询
- 实现
  - 第一种思路：在proxy层的进程代码中实现排序
  - 第二种思路：吧各个分库拿到的数据，汇总到一个MySQL实例的一个表中，然后在这个汇总实例上做逻辑操作

#### 为什么临时表可以重名

- 执行创建临时表语句的时候，MySQL要给这个InnoDB表创建一个frm文件保存表结构定义
  - frm文件放在临时文件目录下，文件名的后缀是`.frm`，前缀`#sql{进程id}_{线程id}_序列号`
- 5.6以前会在临时文件目录创建一个相同前缀以.ibd为后缀的文件，存放数据文件
- 5.7开始，MySQL引入了一个临时文件表空间，专门用来存放临时文件数据。不需要创建.ibd文件
- 根据前缀规则，临时表t1在存储上跟普通表不同

#### 临时表和主备复制

- 如果当前的binlog_format=row，那么跟临时表有关的语句，就不会记录到binlog里。只在binlog_format=statment/mixed 的时候，binlog中才会记录临时表的操作。
- 主库在线程退出的时候，会自动删除临时表，但是备库同步线程是持续在运行的。所以，这时候我们就需要在主库上再写一个`DROP TEMPORARY TABLE`传给备库执行。



## 37讲 什么时候使用内部临时表

#### union执行流程

- `(select 1000 as f) union (select id from t1 order by id desc limit 2);`
  - 取两个子查询结果的并集
-  内存临时表起到了暂存数据的作用，而且计算过程还用上了临时表主键id的唯一性约束，实现了union的语义

#### group by 执行流程

- `select id%10 as m, count(*) as c from t1 group by m;`

#### group by 优化方法 -- 索引

- `alter table t1 add column z int generated always as(id % 100), add index(z);`
- `select z, count(*) as c from t1 group by z;`
- 索引z有序

#### group by 优化方法 -- 直接排序

- 在group by语句中加入SQL_BIG_RESULT这个提示（hint），就可以告诉优化器：这个语句涉及的数据量很大，请直接用磁盘临时表。
- MySQL的优化器一看，磁盘临时表是B+树存储，存储效率不如数组来得高。所以，既然你告诉我数据量很大，那从磁盘空间考虑，还是直接用数组来存吧。
- `select SQL_BIG_RESULT id%100 as m, count(*) as c from t1 group by m;`

不需要执行聚合函数时，distinct 和group by这两条语句的语义和执行流程是相同的，因此执行性能也相同。



## 38讲 要不要使用Memory引擎

#### 内存表的数据组织结构

- Memory引擎的数据和索引是分开的
- 内存表的数据部分以数组的方式单独存放，而主键id索引里，存放的是诶个数据的位置
- 主键id是hash索引，索引上的key并不是有序的

索引组织表：InnoDB引擎把数据放在主键索引上，其他索引上保存的是主键id

堆组织表：Memory引擎把数据单独存放，索引上保存数据位置

#### hash索引和B-Tree索引

- 内存表也支持B-Tree索引
  - `alter table t1 add index a_btree_index using btree (id);`

生产环境不建议使用内存表：

- 锁粒度问题
- 数据持久化问题

#### 内存表的锁

- 内存表不支持行锁，只支持表锁
  - 并发访问的支持不够好

#### 数据持久性问题

- 数据库重启的时候，所有内存表都会被清空

##### 建议把普通内存表都用InnoDB表来代替

- 有一个场景例外：用户临时表。在数据量可控，不会耗费过多内存的情况下，可以考虑使用内存表



## 39讲 自增主键为什么不是连续的

#### 自增值保存在哪儿？

- 表的结构定义存放在后缀名为.frm的文件中，但是并不会保存自增值
- 不同引擎策略不同
  - MyISAM的自增值保存在数据文件中
  - InnoDB的自增值，保存在内存里。8.0后，才有了“自增值持久化”的能力
    - 5.7及以前，自增值保存在内存里，没有持久化，每次重启，第一次打开表时，将max(id)+1作为表当前的自增值
    - MySQL8.0，将自增值的变更记录在`redo log`中

#### 自增值修改机制

- 字段`id`被定义为`AUTO_INCREMENT`。插入一行时：
  - 如果插入数据`id`字段指定为0、null、未指定值，就把当前的`AUTO_INCREMENT`填到自增字段
  - 如果`id`字段制定了具体的值，就直接使用语句里指定的值
- 自增值的变更结果屈居于插入值的大小。假设插入的值X，当前的自增值Y：
  - X<Y，表的自增值不变
  - X>=Y，把当前自增值修改为新的自增值
  - 新的自增值算法：
    - 从`auto_increment_offset`开始，以`auto_increment_increment`为步长，持续叠加，知道直到第一个大于X的值

#### 自增值的修改时机

- 唯一键冲突会致自增主键id不连续
- 事务回滚会导致自增主键不连续
- 批量插入数据会导致自增主键不连续

##### 自增值为什么不能回退？

- MySQL这样设计主要是为了提升性能
- 如果自增值能回退容易造成事务冲突，解决事务冲突会导致性能问题
- InnoDB放弃了自增值回退，保证了自增id递增，但不保证连续

#### 自增锁优化

- 自增id锁并不是一个事务锁，而是每次申请完就马上释放，以便允许别的事务再申请
- `innodb_autoinc_lock_mode`设置为2  `binlog_format`设置为row

## 40讲 insert语句的锁为什么这么多

#### insert ... select 语句

- `insert into t2(c,d) select c,d from t;`
- 可重复读隔离级别下，`binlog_format=statement`时，需要对表t的所有行和间隙加锁
- 原因：考虑日志和数据的一致性

#### insert 循环写入

- 对目标表也不是锁全表，而是值锁住需要访问的资源
- 上面语句会导致在表t上做全表扫描，并且会给索引c上的所有及那次都加上共享的`next-key lock`，所以语句执行期间，其他事务不能在这个表上插入数据

#### insert唯一键冲突

- insert语句发生主键冲突的时候，并不只是简单地报错返回，还在冲突的索引上加了锁（`next-key lock`）
- `insert into … on duplicate key update`
  - 插入一行数据，如果碰到唯一键约束，就执行后面的更新语句（对冲突的行更新）



## 41讲 怎么最快的复制一张表

#### mysqldump方法

- `mysqldump -h$host -P$port -u$user --add-locks --no-create-info --single-transaction  --set-gtid-purged=OFF db1 t --where="a>900" --result-file=/client_tmp/t.sql`
  - `–single-transaction`的作用是，在导出数据的时候不需要对表db1.t加表锁，而是使用`START TRANSACTION WITH CONSISTENT SNAPSHOT`的方法；
  - `–add-locks`设置为0，表示在输出的文件结果里，不增加" LOCK TABLES `t` WRITE;" ；
  - `–no-create-info`的意思是，不需要导出表结构；
  - `–set-gtid-purged=off`表示的是，不输出跟GTID相关的信息；
  - `–result-file`指定了输出文件的路径，其中client表示生成的文件是在客户端机器上的。
- 如果希望生成的文件一行是一条insert语句
  - 在执行mysqldump命令时，加上参数`–skip-extended-insert`。

#### 导出CSV文件

- `select * from db1.t where a>900 into outfile '/server_tmp/t.csv';` 导出

- `load data infile '/server_tmp/t.csv' into table db2.t;` 恢复
- `select …into outfile`方法不会生成表结构文件

#### 物理拷贝方法

- 不可以直接把db1.t表的.frm文件和.ibd文件拷贝到db2的目录下。因为一个InnoDB表，除了包含两个物理文件外，还需要在数据字典中注册
- 可传输表空间（`transportable tablespace`）方法，通过导出+导入表空间的方式，实现物理拷贝表的功能



## 42讲 grant之后要跟着flushprivileges吗

- grant语句是用来给用户赋权的

- `create user 'ua'@'%' identified by 'pa';`这条命令执行两个动作：
  - 磁盘上，往`mysql.user`表里插入一行，由于没有指定权限，所以这行数据上所有表示权限的字段的值都是N；
  - 内存里，往数组`acl_users`里插入一个`acl_user`对象，这个对象的access字段值为0

#### 全局权限

- 全局权限，作用于整个MySQL实例，这些权限信息保存在mysql库的user表里。
- `grant all privileges on *.* to 'ua'@'%' with grant option;`
  - 磁盘上，将mysql.user表里，用户’ua’@’%'这一行的所有表示权限的字段的值都修改为‘Y’；
  - 内存里，从数组acl_users中找到这个用户对应的对象，将access值（权限位）修改为二进制的“全1”。
- grant 命令对于全局权限，同时更新了磁盘和内存。命令完成后即时生效，接下来新创建的连接会使用新的权限。
- 对于一个已经存在的连接，它的全局权限不受grant命令的影响。

- `revoke all privileges on *.* from 'ua'@'%';`  # 回收权限
  - 磁盘上，将mysql.user表里，用户’ua’@’%'这一行的所有表示权限的字段的值都修改为“N”；
  - 内存里，从数组acl_users中找到这个用户对应的对象，将access的值修改为0。

#### db权限

- `grant all privileges on db1.* to 'ua'@'%' with grant option;`
  - 磁盘上，往mysql.db表中插入了一行记录，所有权限位字段设置为“Y”；
  - 内存里，增加一个对象到数组acl_dbs中，这个对象的权限位为“全1”。
- 基于库的权限记录保存在mysql.db表中，在内存里则保存在数组acl_dbs中

#### 表权限和列权限

- 表权限定义存放在表mysql.tables_priv中，列权限定义存放在表mysql.columns_priv中
- `grant all privileges on db1.t1 to 'ua'@'%' with grant option; `
- `GRANT SELECT(id), INSERT (id,a) ON mydb.mytbl TO 'ua'@'%' with grant option;`

`flush privileges`命令会清空`acl_users`数组，然后从`mysql.user`表中读取数据重新加载，重新构造一个`acl_users`数组。

##### 正常情况下，grant命令之后，没有必要跟着执行flush privileges命令。

#### flush privileges使用场景

- 当数据表中的权限数据跟内存中的权限数据不一致的时候
- 这种不一致往往是由不规范的操作导致的
  - 直接用DML语句操作系统权限表



## 43讲 要不要使用分区表

#### 分区表是什么？

- 每个分区对应一个`.ibd`文件
- 对于引擎层来说，是N个表
- 对于Server层来说，是1个表

#### 分区表的引擎层行为

- 分区表和手工分表，一个是由Server层来决定使用那个分区，一个是由应用层代码来决定。从引擎层看，没有差别
- Server层，被广为诟病的问题：打开表的行为

#### 分区策略

- MyISAM分区表使用的分区策略，称为通用分区策略，每次访问分区都由server层控制。有比较严重的性能问题
- MySQL5.7.9开始，InnoDB引擎引入本地分区策略
- MySQL8.0开始不允许创建MyISAM分区表

#### 分区表的server层行为

- 分区表在做DDL的时候，影响会更大

小结：

- 分区表两个问题：
  - MySQL在第一次打开分区表的时候，需要访问所有的分区；
  - 在server层，认为这是同一张表，因此所有分区共用同一个MDL锁；
- 在引擎层，认为这是不同的表，因此MDL锁之后的执行过程，会根据分区表规则，只访问必要的分区。

#### 分区表的应用场景

- 分区表对业务透明，使用分区表业务代码更简洁
- 分区表可以很方便的清理历史数据。
- `alter table t drop partition …`删除分区
  - 速度快，对系统影响小













