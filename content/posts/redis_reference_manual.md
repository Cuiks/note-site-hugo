---
title: "Redis使用手册"
date: 2021-07-06T17:56:38+08:00
tags: ["redis"]
draft: false
---

<!--more-->

### 字符串

#### 1、SET

- 为字符串键设置值

- SET key value
  - SET number "10086" 
  - 返回ok
- SET key value [NX|XX]
  - NX表示对存在的key不能覆盖，对于不存在的可以设置
    - 成功返回ok  失败返回nil
  - XX表示对不存的key不能设置值，对于存在的可以覆盖
    - 成功返回OK 失败返回nil
- O(1)

#### 2、GET

- 获取字符串键的值

- GET key
  - GET number
  - 返回键的值 或者 nil
- O(1)

#### 3、GETSET

- 获取旧值并设置新值

- GETSET key new_value

- O(1)

#### 4、MSET

- 一次为多个字符串键设置值

- MSET key value [key value ... ]
  - MSET message "hello" number "10086"
  - 成功返回ok
- O(n)

#### 5、MGET

- 一次获取多个字符串键的值

- MGET key [key ... ]
  - MGET messgae number page
  - 返回
    - 1) "hello"
    - 2) "10086"
    - 3) nil
- O(n)

#### 6、MSETNX

- 只有在键不存在的情况下，一次为多个字符串设置值

- MSET key value [key value ... ]
  - MSET k1 "one" k2 "two" k3 "three"
  - 只有在所有键都不存在的情况下才会设置
  - 成功返回1。失败返回0
- O(n)

#### 7、STRLEN

- 获取字符串值的字节长度

- STRLEN key
  - STRLEN number
  - 返回key的长度。不存在的key返回0
- 0(1)

#### 8、

- 字符串值的索引

#### 9、GETRANGE

- 获取字符串值指定索引范围上的内容

- GETRANGE key start end
  - GET message
    - "hello world"
  - GETRANGE message 0 3
    - "hell"
  - GETRANGE message -11 -7
    - "hello"
- O(n)

#### 10、SETRANGE

- 对字符串值的指定索引范围进行设置

- SETRANGE key index substitute
  - GET message
    - "hello world"
  - SETRANGE message 6 "Redis"
    - 11  (返回字符串当前长度)
  - GET message
    - "hello Redis"

- SETRANGE命令会自动扩展被修改的字符串值，从而保证新内容可以顺利写入

- SETRANGE命令会在值里面填充空字节
- O(n)

#### 11、APPEND

- 追加新内容到值的末尾

- APPEND key suffix
  - GET desc
    - "Redis"
  - APPEND desc " is a database"
    - 19
  - GET desc
    - "Redis is a database"
- 对于不存的key执行设置操作
- O(n) n为新追加内容的长度

#### 12、

- 使用字符串键存储数字值

#### 13、INCRBY、DECRBY

- 对整数值执行加法操作和减法操作

- INCRBY key increment
  - SET number 100
    - OK
  - INCRBY number 100
    - 400

- DECRBY key increment
  - SET number 100
    - OK
  - DECRBY number 10
    - 90
- 类型限制
  - 当键的值不能被Redis解释为整数时,会返回错误
  - 不能使用除整数外的数据类型作为增量
- 处理不存在的键
  - 对于不存在的键,会先把该键初始化为0,然后操作
- O(1)

#### 14、INCR、DECR

- 对整数值执行加1操作和减1操作

- INCR key

- 类似INCRBY、DECRBY。只是增量固定为1
- O(1)

#### 15、INCRBYFLOAT

- 对数字值执行浮点数加法操作

- 对于浮点数减法操作，增量使用负数
  - INCRBYFLOAT key -1.1
- INCRBYFLOAT可以操作设置为整数的key
- INCRBYFLOAT可以设置增量为整数
- 小数位长度限制
  - 最多保留小数点后17位
- O(1)

### 散列

#### 1、散列简介

#### 2、HSET

- 为字段设置值
- HSET hash field value
  - 如果字段不在散列中，执行创建操作，返回1
  - 如果字段存在散列中，执行更新操作，返回0
  - HSET article title "greeting"
    - 1
  - HSET article content "hello world"
    - 1
  - HSET article count 100
    - 1
  - HSET article title "redis"
    - 0
- O(1)

#### 3、HSETNX

- 只在字段不存在的情况下为他设置值
- HSETNX hash field value
  - 成功返回1
  - 失败返回0
- O(1)

#### 4、HGET

- 获取字段值
- HGET hash field
  - HGET article title
    - "redis"
  - 不存在返回nil
- O(1)

#### 5、HINCRBY

- 对字段存储的整数值执行加法或减法操作
- HINCRBY hash field increment
  - HINCRBY article count 1
    - 101
  - HINCRBY article count -6
    - 95
- O(1)

#### 6、HINCRBYFLOAT

- 对字段存储的数字值执行浮点数加法或减法
- O(1)

#### 7、HSTRLEN

- 获取字段值的字节长度
- HSTRLEN hash field
  - HSTRLEN article title
    - 5  -- redis
  - 字段不存在返回0
  - 散列不存在返回0
- O(1)

#### 8、HEXISTS

- 检查字段是否存在
- HEXISTS hash field
  - 存在返回1
  - 不存在返回0
- O(1)

#### 9、HDEL

- 删除字段
- HDEL hash field
  - 成功返回1
  - hash不存在或者字段不存在返回0
- O(1)

#### 10、HLEN

- 获取散列包含的字段数量
- HLEN hash
  - HLEN article
    - 4
  - 散列不存在返回0
- O(1)

#### 11、HMSET

- 一次为多个字段设置值
- HMSET hash field value [field value ... ]
- O(n)

#### 12、HMGET

- 一次获取多个字段的值
- HMGET hash field [field ... ]
  - 存在返回值
  - 不存在返回nil
- O(n)

#### 13、HKEYS、HVALS、HGETALL

- 获取所有字段、所有值、所有字段和值
- HKEYS hash
- HVALS hash
- HGETALL hash
  - HGETALL article
    - "title"  -- key
    - "greeting"  -- value
    - "content"
    - "hello world"
- 散列不存在
  - 返回空列表
- 字段在散列中存放是无序的
- O(n)  n为散列包含的字段数量

#### 14、

- 散列与字符串

### 列表

- 线性有序
- 文字数据或二进制数据
- 元素可重复

#### 1、 LPUSH

- 将元素推入列表左端
- LPUSH list item [item item ... ]
- 返回列表包含元素数量
- 一次推入多个元素
  - LPUSH todo key1 key2 key3
    - 先推入key1，然后是key2，然后是key3
- O(n)  n为推入元素数量

#### 2、RPUSH

- 将元素推入列表右端
- RPUSH list item [item item ... ]
- 返回列表包含元素数量
- O(n)

#### 3、LPUSHX、RPUSHX

- 只对已存在的列表执行推入操作
- LPUSHX list item
- RPUSHX list item
- 成功返回列表包含元素数量。失败返回0
- 每次只能推入单个元素
- O(1)

#### 4、LPOP

- 弹出列表最左端元素
- LPOP list
- 列表不存在返回nil
- O(1)

#### 5、RPOP

- 弹出列表最右端的元素
- RPOP list
- O(1)

#### 6、RPOPLPSH

- 将右端弹出的元素推入左端
- RPOPLPUSH source traget
- RPOPLPUSH list1 list2
  - 将列表list1的最右端元素弹出，然后推入list2的左端
- 源列表为空则返回nil
- O(1)

#### 7、LLEN

- 获取列表的长度
- LLEN list
- O(1)

#### 8、LINDEX

- 获取指定索引上的元素
- LINDEX list index
- 超出list返回返回nil
- O(n)

#### 9、LRANGE

- 获取指定索引范围上的元素
- LRANGE list start end
  - 返回列表
- 获取列表包含的所有元素
  - LRANGE list 0 -1
- 超出索引范围返回空列表，存在部分则返回部分
- O(n)

#### 10、LSET

- 为指定索引设置新元素
- LSET list index new_element
  - 成功返回1
- 超出索引范围返回错误
  - (error) ERR index out of range
- O(n)

#### 11、LINSERT

- 将元素插入列表某个指定元素的前面或者后面
- LINSERT list BEFORE|AFTER target_element new_element
  - 返回插入后列表长度
  - LINSERT lst BEFORE "b" "10086"
    - 4
- 不存在的元素
  - 返回-1
- O(n)

#### 12、LTRIM

- 修剪列表。移除列表中位于给定索引外的元素
- LTRIM list start end
  - 成功返回ok
- O(n)

#### 13、LREM

- 从列表中删除指定元素
- LREM list count element
  - count=0，删除所有指定元素
  - count>0，从左开始检查，删除count个指定元素
  - count<0，从右向左检查，删除count个指定元素
- O(n)

#### 14、BLPOP

- 阻塞式左端弹出操作。
  - 从左到右依次检查是否有非空列表，没有即阻塞指定时间，直到某个列表不为空或超出等待时间
- BLPOP list [list ... ] timeout
  - 返回两个元素
    - 被弹出元素的来源列表
    - 被弹出元素
  - 如果阻塞过，客户端也会打印阻塞时间
- 阻塞只会阻塞客户端 不影响服务端
- O(n)

#### 15、BRPOP

- 阻塞式右端弹出
- O(n)

#### 16、BRPOPLPUSH

- 阻塞式弹出并推入
- BRPOPLPUSH source target timeout
- O(1)

### 集合

- 可以存储任意多个元素
- 可以存储文本或者二进制数据
- 只存储非重复元素
- 以无序方式存储元素

#### 1、SADD

- 将元素添加到集合
- SADD set element [element ... ]
  - 返回成功添加元素的数量
- 对于已存在的元素，不会添加到集合中，返回0
- O(n)  n为添加元素的数量

#### 2、SREM

- 从集合中移除元素
- SREM set element [element ... ]
  - 返回被移除元素数量
- O(n) n为元素给定元素数量

#### 3、SMOVE

- 将元素从一个集合移动到另一个集合
- SMOVE source target element
  - 成功返回1
  - 元素不存在返回0
- O(1)

#### 4、SMEMBERS

- 获取集合包含的所有元素
- SMEMBERS set
- 返回元素无序
- O(n)

#### 5、SCARD

- 获取集合包含的元素数量
- SCARD set
- O(1)

#### 6、SISMEMBER

- 检查给定元素是否存在于集合中
- SISMEMBER set element
  - 存在返回1
  - 不存在返回0
- O(1)

#### 7、SRANDMEMBER

- 随机获取集合中的元素
- SRANDMEMBER set [count]
  - count默认为1
  - 随机返回count个元素
- O(n) n为count的数量

#### 8、SPOP

- 随机地从集合中移除指定数量的元素
- SPOP set [count]
  - count 默认为1
  - 返回count个元素
- O(n)  n为count的数量

#### 9、SINTER、SINTERSTORE

- 对集合执行交集计算
- SINTER set [set ... ]
  - 返回n个集合的交集
  - O(m*n)
- SINTERSTORE destination_key set [set ... ]
  - 把n个集合的交集存储到destination_key中
  - 返回交集长度
  - O(m*n)

#### 10、SUNION、SUNIONSTORE

- 对集合执行并集运算
- SUNION set [set ... ]
  - 返回n个集合的并集
  - O(n) n为所有集合元素的总数
- SUNION destination_key set [set ... ]
  - 把n个集合的并集存储到destination_key中
  - 返回并集长度
  - O(n) n为所有集合元素的总数

#### 11、SDIFF、SDIFFSTORE

- 对集合执行差集运算
- SDIFF set [set ... ]
  - 返回n个集合的差集
  - O(n)
- SDIFFSTORE destination_key set [set ... ]
  - 把n个集合的差集保存到destination_key中
  - 返回差集的长度
  - O(n)

### 有序集合

- 有序
- 集合
- 每一个元素由成员和成员分值组成

#### 1、ZADD

- 添加或更新成员
- ZADD sorted_set score member [socre member ... ]
  - 返回新成员数量
  - ZADD salary 3500 "peter" 4000 "jack" 2000 "tom" 5500 "mary"
    - 返回 4
  - ZADD salary 5000 "tom"
    - 返回0  因为没有添加新成员
- ZADD sorted_set [XX|NX] score member [socre member ... ]
  - XX：只对已存在的成员更新，不会添加
  - NX：只对不存在的成员添加，不会更新
- ZADD sorted_set [CH] score member [socre member ... ]
  - 返回被修改的成员数量（包括新增和被更新）
- O(m*log(n)) m为给定成员的数量。n为有序集合包含的成员数量

#### 2、ZREM

- 移除指定的成员
- ZREM sorted_set member [member ... ]
  - 返回被移除成员的数量
- O(M*log(n))  m为给定成员数量，n为有序集合包含的成员数量

#### 3、ZSCORE

- 获取成员的分值
- ZSCORE sorted_set member
- O(1)

#### 4、ZINCRBY

- 对成员的分值执行自增或自减操作
- ZINCRBY sorted_set increment member
  - 返回修改后的分值
  - ZINCRBY salary 1000 "tom"
    - "3000"
  - ZINCRBY salary -1000 "tom"
    - "2000"
- 对于不存在的成员
  - 添加成员并设置分值
- 对于不存在的集合
  - 自动创建集合
- O(log(n))

#### 5、ZCARD

- 获取有序集合的大小
- ZCARD sorted_set
  - 返回长度
- O(1)

#### 6、ZRANK、ZREVRANK

- 获取成员在有序集合中的排名
- ZRANK sorted_set member
  - 按照分值从小到大排序返回排名
  - 从0开始
- ZREVRANK sorted_set member
  - 按照分值从大到小排序返回排名
  - 从0开始
- 对于不存在的键和或者不存在的成员
  - 返回nil
- O(log(n))

#### 7、ZRANGE、ZREVRANGE

- 获取指定索引范围内的成员
- ZRANGE sorted_set start end
  - 分值升序返回
- ZREVRANGE sorted_set start end
  - 分值降序返回
- ZRANGE sorted_set start end [WITHSOCRES]
  - 返回成员和分值
- 不存在的有序集合，返回空列表或集合
- O(log(n) + m)

#### 8、ZRANGEBYSCORE、ZREVRANGEBYSCORE

- 获取指定分值范围内的成员
- ZRANGEBYSCORE sorted_set min max
  - 升序排列方式返回
  - [min, max]闭区间
- ZREVRANGEBYSCORE sorted_set max min
  - 降序排列方式返回
- ZRANGEBYSCORE sorted_set min max [WITHSCORES]
  - 带分值返回
- ZRANGEBYSCORE sorted_set min max [LIMIT offset count]
  - offser指定跳过成员数量。count只是返回成员个数
  - ZRANGEBYSCORE salary 3000 5000 0 1
    - "peter"
- 使用开区间取值 "("
  - ZRANGEBYSCORE salary (3500 (5000 WITHSCORES
    - "bob"
    - "3800"
    - "jack"
    - "4500"
  - ZRANGEBYSCORE salary 3500 (5000 WITHSCORES
    - "peter"
    - "3500"
    - "bob"
    - "3800"
    - "jack"
    - "4500"
- 使用无限制作为范围
  - ZRANGEBYSCORE salary -inf (5000 WITHSCORES
- O(log(n) + m)

#### 9、ZCOUNT

- 统计指定分值范围内的成员数量
- ZCOUNT sorted_set min max
  - [min, max]闭区间
  - 返回成员个数
- "+inf"表示无穷大
- "-inf"表示无穷小
- "("表示开区间
- O(log(n))

#### 10、ZREMRANGEBYRANK

- 移除指定排名范围内的成员
- ZREMRANGEBYRANK sorted_set start end
  - [start, end]闭区间
  - 返回移除成员数量
- O(log(n) + m)

#### 11、ZREMRANGEBYSCORE

- 移除指定分值范围内的成员
- ZREMRANGEBYSCORE sorted_set min max
  - [min, max]闭区间
  - 返回移除成员个数
- O(log(n) + m)

#### 12、ZUNIONSTORE、ZINTERSTORE

- 有序集合的并集运算和交集运算
- ZUNIONSTORE destination number sorted_set [sorted_set ... ]
  - number参数用于指定参与计算的有序集合数量
  - 计算并集存储至destination
  - 返回并集长度
  - O(n*log(n))
- ZINTERSTORE destination number sorted_set [sorted_set ... ]
  - number参数用于指定参与计算的有序集合数量
  - 计算交集存储至destination
  - 返回交集长度
  - O(n\*log(n)\*m)
- 指定分数的聚合函数
  - ZUNIONSTORE destination number sorted_set [sorted_set ... ] [AGGREGATE SUM|MIN|MAX]
    - SUM：所有集合的分值的和
    - MIN：所有集合中分值最小的值
    - MAX：所有集合中分值最大的值
- 为每个有序集合设置权重
  - 权重会与分值相乘，然后执行聚合运算
  - ZUNIONSTORE destination number sorted_set [sorted_set ... ] [WEIGHTS weight [weight ... ]]
- 使用集合作为输入
  - 这两个命令也可以操作集合。默认分值为1

#### 13、ZRANGEBYLEX、ZREVRANGEBYLEX

- 返回指定字典序范围内的成员
- ZRANGEBYLEX sorted_set min max
  - "["表示包含
  - "("表示不包含
  - "+"表示无穷大
  - "-"表示无穷小
  - ZRANGEBYLEX words - +
    - 返回所有
  - ZRANGEBYLEX words [a (b
    - 返回a开头的所有  不返回b开头的
- ZREVRANGEBYLEX sorted_set max min
  - 倒序返回
- ZRANGEBYLEX sorted_set min max [LIMIT offset count]
  - 限制返回数量
- O(log(n) + n)

#### 14、ZLEXCOUNT

- 统计位于字典序指定范围内的成员数量
- ZLEXCOUNT sorted_set min max
  - "[" "(" "+" "-"
- O(nlog(n))

#### 15、ZREMRANGEBYLEX

- 移除位于字典序指定范围内的成员
- ZREMRANGEBYLEX sorted_set min max
  - 返回移除元素数量
- O(log(n) + m)

#### 16、ZPOPMAX、ZPOPMIN

- 弹出分值最高和最低的成员
- ZPOPMAX sorted_set [count]
  - count默认为1
  - 返回被移除的成员和分值
- ZPOPMIN sorted_set [count]
  - count默认为1
  - 返回被移除的成员和分值
- O(n) n为移除元素数量

#### 17、BZPOPMAX、BZPOPMIN

- 阻塞式最大/最小元素弹出操作
- BZPOPMAX sorted_set [sorted_set ... ] timeout
  - 返回被弹出元素所在集合。成员。分值
- BZPOPMIN sorted_set [sorted_set ... ] timeout
- O(n)



### hyperloglog

#### 1、

hyperloglog是专门为了计算集合的基数而创建的概率算法

- 对集合的元素进行基数
- 获取集合当前的近似基数
- 合并多个HyperLogLog

#### 2、PFADD

- 对集合元素进行计数
- PFADD hyperloglog element [element ... ]
  - 若基数没变返回0
  - 若基数变化返回1
- O(n)

#### 3、PFCOUNT

- 返回集合的近似基数
- PFCOUNT hyperloglog [hyperloglog ... ]
  - 返回近似基数
  - 不存在返回0
- 返回并集的近似基数
  - 传入多个HyperLogLog时，将进行并集运算，返回并集的近似基数
- O(n)

#### 4、PFMERGE

- 计算多个HyperLogLog的并集
- PFMERGE destination hyperloglog [hyperloglog ... ]
  - 并集保存到destination
  - 如果指定的键已存在则覆盖
  - 成功返回ok
- PFCOUNT命令会调用PFMERGE获取临时HyperLogLog，然后计算近似基数，删除临时HyperLogLog
- O(n)

### 位图

#### 1、SETBIT

- 设置二进制的值
- SETBIT bitmap offset value
  - 返回二进制位被设置前的旧值
  - 偏移量只能为正数
- 位图不存在将被创建
- 位图长度不够将被扩展
- O(1)

#### 2、GETBIT

- 获取二进制位的值
- GETBIT bitmap offset
  - 返回偏移量上的二进制值
  - 偏移量只能为整数
  - 对于偏移量范围外的返回0
- O(1)

#### 3、BITCOUNT

- 统计被设置的二进制位数量
- BITCOUNT key
  - 返回被设置为1的位数
- BITCOUNT key [start end]
  - 在指定字节范围统计
  - start，end为字节偏移量
  - 1字节 = 8bit
  - 1字节偏移量 = 8bit位偏移量
- O(n)

#### 4、BITPOS

- 查找第一个指定的二进制位值
- BITPOS bitmap value
  - 返回第一个被设置为value的位置偏移量
- BITPOS bitmap value [start end]
  - 在指定字节范围搜索第一个被设置value的位置
- O(n)

#### 5、BITOP

- 执行二进制位运算
- BITOP operation result_key bitmap [bitmap ... ]
  - opetration: AND、OR、XOR、NOT
  - NOT只允许输入一个位图，其他允许多个
  - 返回被存储位图的字节长度
- 两个不同长度的位图，较短的会在前补0
- O(n)

#### 6、BITFIELD

- 在位图中存储整数值
- BITFIELD支持SET、GET、INCRBY、OVERFLOW四个子命令
- BITFIELD bitmap SET type offset value
  - type：i8(有符号8位整数) u16(无符号16位整数)
  - BITFIELD bitmap SET u8 0 198
    - 0
  - BITFIELD bitmap SET u8 0 123 SET i32 20 10086 SET i64 188 123456789
    - 198
    - 0
    - 0
- O(n)

#### 7、

- TYPE bitmap
  - string

### 地理坐标

#### 1、GEOADD

- 存储坐标
- GEOADD location_set longitude latitude name [ longitude latitude name ... ]
  - 返回添加个数
- GEOADD可以更新已存在的地理位置
  - 更新返回0
- O(log(n)*m)

#### 2、GEOPOS

- 获取指定位置坐标
- GEOPOS location_set name [name ... ]
  - 返回n组数组
  - GEOPOS guangdong qingyuan guangzhou 
    - 1) 1) "113"  -- 清远市经度
    - ​    2) "23"   -- 清远市纬度
    - 2) 1) "113"  -- 广州市经度
    - ​    2)  "23"   -- 广州市纬度
    - 不存在返回nil
- O(log(n)*m)

#### 3、GEODIST

- 计算两个位置之间的直线距离
- GEODIST location_set name1 name2
  - 返回name1和name2的距离
- GEODIST location_set name1 name2 [unit]
  - unit可选：m(米),km(千米),mi(英里),ft(英尺)
- 不存在的位置返回nil
- O(log(n))

#### 4、GEORADIUS

- 查找指定坐标半径范围内的其他位置
- GEORADIUS location_set longitude latitude radius unit
  - GEORADIUS guangdong 112 23 100 km
    - "foshan"
    - "guangzhou"
- GEORADIUS location_set longitude latitude radius unit [WITHDIST]
  - WITHDIST: 返回携带距离
- GEORADIUS location_set longitude latitude radius unit [WITHCOORD]
  - WITHCOORD：返回携带经纬度
- GEORADIUS location_set longitude latitude radius unit [ASC|DESC]
  - 排序查找
- GEORADIUS location_set longitude latitude radius unit [COUNT n]
  - 限制返回的数量
- O(n)

#### 5、GEORADIUSBYMEMBER

- 查找指定位置半径范围内的其他位置
- GEORADIUSBYMEMBER location_set name radius unit [WITHDIST] [WITHCOORD] [ASC|DESC] [COUNT n]
- O(n)

#### 6、GEOHASH

- 获取指定位置的Geohash值
- GEOHASH guangdong qingyuan guangzhou
- GEORADIUS、GEORADIUSBYMEMBER可以添加[WITHHASH]参数返回Geohash
- O(n)

#### 7、

- 可以使用ZREM对location中位置进行删除

### 流

- redis5.0新增

#### 1、XADD

- 追加新元素到流的末尾
- XADD stream id field value [field value ... ]
  - 流不存在则创建
  - id由"毫秒时间-编号"两部分组成
    - XADD stream 1000000-123 k1 v1 
  - 返回编号id
  - id不可以重复
  - 新id必须大于旧id
  - XADD s1 * k1 v1
    - 自动生成id
- 限制流的长度
  - XADD stream [MAXLEN len] id field value [field value ... ]
  - 按照先进先出移除超过长度的元素
- O(log(n))

#### 2、XTRIM

- 对流进行修剪。将流修剪为指定长度
- XTRIM stream MAXLEN len
  - 返回被移除元素数量
- O(log(n)+m) n为修剪操作前元素数量。m为被修剪元素数量

#### 3、SDEL

- 移除指定元素
- XDEL stream [id id ... ]
  - 返回被移除元素数量
- O(log(n)*m)

#### 4、XLEN

- 获取流包含的元素数量
- XLEN stream
- O(1)

#### 5、XRANGE、XREVRANGE

- 访问流中元素
- XRANGE stram start-id end-id [COUNT n]
  - 返回ID1
    - k1
    - v1
  - ...
- XRANGE temp 4000000000000 + 
  - 获取所有大于4000000000000的
- XRANGE temp - 4000000000000 
  - 获取所有小于4000000000000的
- COUNT 限制获取的数量
- XREVRANGE stram start-id end-id [COUNT n]
  - 逆序访问
- O(log(n)+m)

#### 6、XREAD

- 以阻塞或非阻塞方式获取流元素
- XREAD [BLOCK ms] [count n] STREAMS stream1 stream2 stream3 ... id1 id2 id3 ...
  - 从给定的流中取出大于指定ID的多个元素
  - XREAD COUNT 3 STREAMS s1 10000000
- XREAD BLOCK ms STREAMS stream1 stream2 ... $ $ $ ...
  - 只获取新元素
  - XREAD BLOCK 10000 STREAMS bs1 $
    - 只获取bs1后续出现的新元素
- O((log(n)+m)*I) 流包含n个元素。m为被获取元素数量。I个流

#### 7、XGROUP、XREADGROUP、XACK

- 消费者组
- XGROUP CREATE stream group start_id
  - 创建消费者组
  - group为组名
  - start_id为消费者开始读取的id
- XREADGROUP GROUP consumer [COUNT n] [BLOCK ms] STREAMS stream [stream ... ] id [id ... ]
  - 读取消费者组
  - XREADGROUP GROUP g1 c1 STREAMS msgs 0
- XACK stream group id [id id ...]
  - 消息的状态转换

#### 8、XGROUP

- 管理消费者组
- XGROUP CREATE stream group id
  - 创建消费者组
  - 如果stream不存在返回错误
  - XGROUP CREATE cgs all-message 0-0
  - XINFO GROUP cgs
  - O(1)
- XGROUP SETID stream group id
  - 修改消费者组最后递送消息ID
  - O(1)
- XGROUP DELCONSUMER stream group consumer
  - 删除消费者
  - O(n) n为被删除消费者正在处理的消息数量
- XGROUP DESTORY stream group
  - 删除消费者组
  - O(n+m) n为消费者组被删除时，待处理的消息数量。m为消费者数量

#### 9、XREADGROUP

- 读取消费者组中的消息
- XREADGROUP GROUP group consumer [COUNT n] [BLOCK ms] STREAMS stream [stream ...] id [id ...]
- XREADGROUP GROUP all-message worker1 COUNT 1 STREAMS cgs >
  - 读取未递送过的新消息
- O((log(n)+m)*i)

#### 10、XPENDING

- 显示待处理消息的相关信息
- XPENDING stream group [start stop count] [consumer]
- O(log(n)+m)

#### 11、XACK

- 将消息标记为“已处理”
- XACK stream group id [id id ...]
- O(n)

#### 12、XCLAIM

- 转移消息的归属权
- XCLAIM stream group new_consumer max_pending_time id [id id ...]
- 只转移消息ID
  - XCLAIM stream group new_consumer max_pending_time id [id id ...] [JUSTID]
- O(n)

#### 13、XINFO

- 查看流和消费者组的相关信息
- XINFO CONSUERS stream group-name
  - 打印消费者信息
  - O(n) n为消费者数量
- XINFO GROUPS stream
  - 打印消费者组的信息
  - O(m) m为消费者组数量
- XINFO STREAM stream
  - 打印流信息
  - O(1)



### 数据库

#### 1、SELECT

- 切换至指定的数据库
- SELECT db
  - SELECT 0
- O(1)

#### 2、KEYS

- 获取所有与给定匹配符相匹配的键
- KEYS pattern
  - KEYS *（阻塞服务器）
- O(n)

#### 3、SCAN

- 以渐进方式迭代数据库中的键
- SCAN cursor
  - SCAN 0
- SCAN命令不需要申请也不需要释放，不占用任何资源
- SCAN cursor [MATCH pattern]
  - 迭代与给定匹配符相匹配的键
- SCAN cursor [COUNT n]
  - 返回指定数量
- Redis中可能导致阻塞的命令
  - KEYS
  - HKEYS、HVALS、HGETALL
  - SMEMBERS
  - ZRANGE
  - 为解决以上问题提供了SSCAN、HSCAN、ZSCAN
- HSCAN hash cursor [MATCH pattern] [COUNT n]
- SSCAN set cursor [MATCH pattern] [COUNT n]
- ZSCAN sorted_set cursor [MATCH pattern] [COUNT n]
- 单次调用O(1)。迭代一次O(n)

#### 4、RANDOMKEY

- 随机返回一个键
- RANDOMKEY
- O(1)

#### 5、SORT

- 对键的值进行排序
- SORT key
- 指定排序方式
  - SORT key [ASC|DESC]
- 对字符串进行排序
  - SORT key [ALPHA]
- 只获取部分排序结果
  - SORT key [LIMIT offset count]
- 获取外部键的值作为结果
  - SORT key [[GET pattern] [GET pattern] ...]
- 使用外部键的值作为排序权重
  - SORT key [BY pattern]
- 保存排序结果
  - SORT key [STORE destination]
- O(n*log(n)+m)

#### 6、EXISTS

- 检查给定键是否存在
- EXISTS key [key ...]
  - 返回存在的key的数量
- O(n)

#### 7、DBSIZE

- 获取数据库中包含的键值对数量
- DBSIZE
- O(1)

#### 8、TYPE

- 查看键类型

- TYPE key

- | 键类型      | TYPE返回值 |
  | ----------- | ---------- |
  | 字符串键    | string     |
  | 散列键      | hash       |
  | 列表键      | list       |
  | 集合键      | set        |
  | 有序集合键  | zset       |
  | HyperLogLog | string     |
  | 位图        | string     |
  | 地理位置    | zset       |
  | 流          | stream     |

  O(1)

#### 9、RENAME、RENAMENX

- 修改键名
- RENAME origin new
  - new已存在则会覆盖
- RENAMENX origin new
  - new已存在则不覆盖
- O(1)

#### 10、MOVE

- 将给定的键移动到另一个数据库
- MOVE key db
- 源数据库不存在key或者目标数据库存在key该操作都会失败
- O(1)

#### 11、DEL

- 移除指定的键
- DEL key [key ...]
  - 返回被移除的键的数量
- O(n)

#### 12、UNLINK

- 以异步方式移除指定的键（防止阻塞）
- UNLINK key [key ...]
  - 返回成功移除的键的数量

#### 13、FLUSHDB

- 清空当前数据库
- FLUSHDB
  - ok
- FLUSHDB async
  - 异步方式清除数据库
- O(n)

#### 14、FLUSHALL

- 清空所有数据库
- FLUSHALL
  - ok
- FLUSHALL async
  - 异步方式
- O(n)

#### 15、SWAPDB

- 互换数据库
- SWAPDB x y
- O(1)

### 自动过期

#### 1、EXPIRE、PEXPIRE

- 设置生存时间
- EXPIRE key seconds
  - 秒级过期时间
- PEXPIRE key milliseconds
  - 毫秒级过期时间
- O(1)

#### 2、SET-EX、PX

- SET key value [EX seconds] [PX milliseconds]
- O(1)

#### 3、EXPIREAT、PEXPIREAT

- 设置(更新)过期时间
- EXPIREAT key seconds_timestamp
  - 超过给定的秒级时间戳，key被删除
- PEXPIRAET key milliseconds_timestamp
  - 超过给定的毫秒级时间戳，key被删除
- O(1)

#### 4、TTL、PTTL

- 获取键的剩余生存时间
- TTL key
- PTTL key
- 没有过期时间的key返回-1
- 不存在的key返回-2
- O(1)

### 流水线与事物

#### 1、流水线

- 允许客户端打包多条redis请求一次性发送给服务器，处理完毕后一次性返回全部处理结果

#### 2、事务

- MULTI
  - 开启事务
  - O(1)
- EXEC
  - 执行事务
- DISCARD
  - 放弃事务
  - O(n)
- 事务会独占服务器。避免事务执行过多命令造成服务器阻塞。

#### 3、带有乐观锁的事务

- WATCH
  - 对键进行监视
  - WATCH key [key ...]
  - 确保事务只会在被监视键没有发生变化的情况下执行，从而保证事务对被监视键的所有修改都是安全、正确和有效的。
  - O(n)
- UNWATCH
  - 取消对键的监视
  - UNWATCH key [key ...]
  - O(n)

### LUA脚本

- lua脚本执行具有原子性

#### 1、EVAL

- 执行脚本
- EVAL script numkeys key [key ...] arg [arg ...]
- 使用脚本执行redis命令
  - redis.call(command, ...)
    - redis.call('SET', KEYS[1], ARGV[1])
  - redis.pcall(command, ...)

#### 2、SCRIPT LOAD和EVALSHA

- 缓存并执行脚本
- ACRIPT LOAD script
  - 缓存脚本
- EVALSHA sha1 numkeys key [key ...] arg [arg ...]
  - 执行脚本

#### 3、脚本管理

- SCRIPT EXISTS sha1 [sha1 ...]
  - 脚本是否已存在。存在返回1，不存在返回0
- SCRIPT FLUSH
  - 移除所有已缓存脚本
- SCRIPT KILL 
  - 强制停止正在运行的脚本
  - kill失败只能 "SHUTDOWN nosave"关闭服务器
- O(1)

#### 4、内置函数库

- redis包
- bit包
- struct包
- cjson包
- cmsgpack包

### 持久化

- RDB持久化
- AOF持久化
- RDB-AOF混合持久化

#### 1、RDB持久化

- Redis默认
- 全量式持久化
- 生成.rdb结尾文件(Redis DataBase)
- SAVE
  - 阻塞服务器并创建RDB文件
  - 返回OK
  - 如果已经存在RDB文件 SAVE会移除就文件，只保留新的RDB文件
  - O(n)  n为键值对总数量
- BGSAVE
  - 以非阻塞方式创建RDB文件
  - 使用子进程创建RDB文件
  - O(n)  n为键值对总数量
- 通过配置自动创建RDB文件
  - save <seconds> <changes>
    - 服务器在seconds秒内执行了changes次自改，自动BGSAVE。
  - Redis默认
    - save 60 10000
    - save 300 100
    - save 3600 1
- RDB持久化缺陷
  - 间隔越长，停机时丢失数据越多
  - 全量持久化操纵消耗大量计算资源和内存资源

#### 2、AOF持久化

- 增量式持久化
- 服务器每次执行完写命令之后，都会以协议文本方式将被执行命令最佳到AOF文件末尾
- 打开AOF持久化功能
  - appendbonly [yes | no]
    - appendonly yes  # 开启
    - appendonly no  # 关闭
- 设置AOF文件的冲洗频率(把内存缓冲区数据写入文件)
  - appendfsync [always | everysec | no]
    - always：每次执行写命令
    - everysec：每隔1s  （默认选项）
    - no：不主动冲洗
- AOF重写（优化AOF历史命令，只保存恢复当前数据库所需的尽可能少的命令）
  - BGRERITEAOF
    - 显式触发AOF重写操作
    - 异步命令
    - 如果正在创建RDB文件，将会等待。避免两个写磁盘造成性能下降
    - 执行中，重复收到该命令将返回错误
    - O(n)
  - auto-aof-rewrite-min-size <value>
    - auto-aof-rewrite-min-size 64m
    - 当AOF文件大于value时，将会自动执行重写
  - auto-aof-rewrite-percentage <value>
    - auto-aof-rewrite-percentage 100
    - 当前AOF文件体积比最后一次AOF重写之后打了一倍(100%)后重写
- AOF持久化优点
  - 比RDB安全性高得多(丢失数据少)
- AOF持久化缺点
  - 存储协议文本比RDB二进制大得多。生成时间也更长
  - AOF需要执行命令来恢复。恢复速度更慢
  - AOF重写会占用大量资源。创建子进程会造成短暂阻塞

#### 3、RDB-AOP混合持久化

- AOF文件有RDB数据和生成RDB之后的命令组成
- aof-use-rdb-preamble <value>
  - yes: Redis在执行AOF重写操作时，会像执行BGSAVE，同事生成RDB数据写入AOF文件。对于新执行命令继续以协议文本方式写入AOF文件。
  - no：redis默认

#### 4、

#### 5、无持久化

- 关闭系统默认持久化
  - save '' ''

#### 6、关闭服务器

- SHUTDOWN
- SHUTDOWN [save | nosave]
  - 关闭前进行一次持久化操作
  - 开启RDB会执行SAVE命令。
  - 开启AOF会冲洗AOF文件。
- O(n)  n为需要持久化的键值对数量



### 发布与订阅

#### 1、PUBLISH

- 向频道发送消息
- PUBLISH channel message
  - PUBLISH "news.it" "hello"
- O(n+m) n=订阅者数量。m=被订阅的模式总数量

#### 2、SUBSCRIBE

- 订阅频道
- SUBSCRIBE channel [channel ... ]
  - SUBSCRIBE "new.it"
- O(n)  n为订阅频道数量

#### 3、UNSUBSCRIBE

- UNSUBSCRIBE [channel channel ... ]
- O(n)

#### 4、PSUBSCRIBE

- 订阅模式
- PSUBSCRIBE pattern [pattern ... ]
  - PSUBSCRIBE "news.*"
- O(n)  n为给定模式数量

#### 5、PUNSUBSCRIBE

- 退订模式
- O(n*m) n=给定模式数量 m=被订阅模式数量

#### 6、PUBSUB

- 查看发布与订阅的相关信息
- PUBSUB CHANNELS
  - 查看被订阅的频道
- PUBSUB NUMSUB [channel channel ...]
  - 查看频道的订阅者数量
- PUBSUB NUMPAT
  - 查看被订阅模式的总数量
- O(1)



### 模块

通常由C语言或其他能够与C语言交互的语言实现

#### 1、模块的管理

- 编译模块
- 载入模块
  - 在服务启动时
    - loadmoule <module_path>
  - 在服务运行期间
    - module load module_path
  - O(1)
- 列出已载入的模块
  - MODULE LIST
  - O(n) n为模块数量
- 卸载模块
  - MODULE UNLOAD module_name
  - O(1)

#### 2、ReJSON模块

- MODULE LOAD ~/src/rejson.so
- JSON.SET <key> <path> <json> [NX|XX]
- JSON.GET <key> [path path ...]
- JSON.DEL <key> [path]
- JSON.MGET <key> [key ...] <path>
- JSON.TYPE <key> [path]

#### 3、RediSQL模块

- REDISQL CREATE_DB name [path]
  - 创建数据库
  - O(1)
- DEL db [db ... ]
  - 删除数据库
- REDISQL.EXEC db "statement"
  - 执行语句
  - REDISQL.EXEC MYDB "INSERT INTO user VALUES('peter')"
- REDISQL.QUERY db "statement"
  - 执行查询
- REDISQL.COPY source destination
  - 复制数据库

#### 4、RediSearch模块

- FT.CREATE name SCHEMA [field [WEIGHT n] [TEXT | NUMERIC | GEO |TAG] [SORTABLE] ] [FIELD ...]
  - 创建索引
- FT.ADD index docId score FIELDS field value [field value]
  - 添加文档至索引
- FT.ADDHASH index hash score [LANGUAGE lang] [REPLACE]
  - 添加散列至索引
- FT.INFO index
  - 查看索引信息
- FT.SEARCH index query [LANGUAGE lang]
  - 检索文档
- FT.GET index docId
  - FT.MGET index docId [docId ...]
  - 从索引中获取文档
- FT.DEL index docID
  - 从索引中删除文档
- FT.DROP index [KEEPDOCS]
  - 移除索引

### 复制

- 主从复制

#### 1、REPLICAOF

- 将服务器设置为从服务器
- REPLICAOF host port
  - 成为host:port 服务的从服务器
- 在启动的同时设置为从服务器
  - replicaof <host> <port>
    - redis-server --port 10086 --replicaof 127.0.0.1 6379
- 取消复制
  - REPLICAOF no one
  - 不会清空数据，继续保留复制产生的所有数据

#### 2、ROLE

- 查看服务器的角色
- ROLE
  - master  # 主服务器
  - slave  # 从服务器
- O(1)

#### 3、数据同步

- 执行REPLICAOF后：
  - 主服务器执行BGSAVE生成RDB文件，并使用缓冲区存储之后执行的所有写命令
  - 通过套接字吧RDB传送给从服务器
  - 从服务器接受RDB文件，载入
  - 接受主服务器缓存区所有写命令
- 在线更新
  - 主服务器执行完写命令后会把相同写命令或具有相同效果的写命令发给从服务器执行
- 部分同步
  - 因故障下线的从服务器重新上线时
  - 主服务维护命令队列。如果从服务器上线时缺失的命令存在于队列则执行队列中的命令更新，否则执行完整同步
  - 通过repl-backlog-size 修改队列大小

#### 4、无需硬盘的复制

- 创建RDB文件导致复制变慢
- repl-diskless-sync <yes|no>
  - redis-server repl-diskless-sync yes
  - 不在本地创建RDB文件。创建子进程直接通过套接字将RDB文件写入从服务器

#### 5、降低数据不一致情况出现的概率

- min-replicas-max-lag <seconds>
- min-relicas-to-write <numbers>
  - 主服务器只会在从服务器数量大于min-relicas-to-write且从服务器与主服务器最后一次成功通信的间隔不超过min-replicas-max-lag时才会执行写命令

#### 6、可写的从服务器

- 从服务器默认只允许读
- replica-read-only <yes|no>
  - 设置no，表示可写

#### 7、脚本复制

- 脚本传播模式
- 命令传播模式
- 选择性命令传播



### Sentinel

- 故障转移：使用正常服务器替换下线服务器以维持系统正常运行
- Redis Sentinel通过心跳检测的方式监视多个主服务器以及它们属下从服务器，并在某个主服务器下线时自动对其实施故障转移

#### 1、启动Sentinel

- redis-sentinel /etc/sentinel.conf
  - sentinel monitor <master-name> <ip> <port> <quorum>  # 配置
  - ...
- 对于重新上线的旧主服务器会被转换为当前主服务器的从服务器
- replica-priority配置选项可以设置从服务器优先级
- 选主规则
  - 优先级高
  - 偏移量大
  - 运行ID小

#### 2、Sentinel网络

#### 3、Sentinel管理命令

- SENTINEL masters
  - 获取所有被监视主服务器的信息
- SENTINEL master <master-name>
  - 获取指定主服务器的信息
- SENTINEL slaves
  - 获取所有被监视主服务器的从服务器信息
- SENTINEL sentinels
  - 获取其他Sentinel的相关信息
- SENTINEL get-master-addr-by-name <master-name>
  - 获取给定主服务器的IP地址和端口号
- SENTINEL reset <pattern>
  - 重置主服务器状态
- SENTINEL failover <master-name>
  - 强制执行故障转移
- SENTINEL ckquorum <master-name>
  - 检查可用Sentinel数量
- SENTINEL flushconfig
  - 强制写入配置文件

#### 4、在线配置Sentinel

- SENTINEL monitor <master-name> <ip> <port> <quorum>
  - 监视给定主服务器
- SENTINEL remove <master-name>
  - 取消对给定主服务器的监视
- SENTINEL set <master-name> <option> <value>
  - 修改Sentinel配置选项的值



### 集群

#### 1、基本特性

- 复制与高可用
- 分片与重分片
- 高性能
- 简单易用

#### 2、搭建集群

- 快速搭建集群
  - create-cluster
- 手动搭建集群

#### 3、散列标签

#### 4、打开/关闭从节点的读命令执行权限

- READONLY
  - 打开读命令执行权限
- READWRITE
  - 关闭读命令执行权限
- O(1)

#### 5、集群管理工具redis-cli

- redis-cli --cluster help

#### 6、集群管理命令



