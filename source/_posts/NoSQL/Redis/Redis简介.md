---
title: Redis简介
date: 2022/7/14 20:46:25
tags:
- NoSQL
- Redis
categories:
- [NoSQL, Redis]
description: Redis简介
---

# Redis基本信息

## NoSQL特点

1. `NoSQL`：即 `Not-Only SQL` 泛指非关系型的数据库，作为关系型数据库的补充
2. 作用：应对基于海量用户和海量数据前提下的数据处理问题
3. 特点：可扩容、可伸缩、大数据量下高性能、灵活的数据模型、高可用

## Redis基本信息

1. 基本：端口号 `6379`，最大存储值 `512MB`
2. 使用场景：热点信息，波段性（不是一直都是热点）
3. 概念：`Redis` (**RE**mote **DI**ctionary **S**erver) 是 `C ` 语言开发的一个开源的**高性能**键值对 `key-value` 数据库
4. 特点
    1. 数据间没有必然的关联关系
    2. 内部采用单线程机制进行工作
    3. 高性能。官方提供测试数据，50个并发执行100000个请求,读的速度是110000 次/s,写的速度是81000次/s。
    4. 多数据类型支持：字符串 `String`、列表 `List`、集合 `Set`、有序集合 `ZSet(SortedSet)`、散列类型 `Hash`
    5. 持久化支持，可以进行数据灾难恢复
5. 应用场景
    1. 为热点数据加速查询（主要场景），如热点商品、热点新闻、热点资讯、推广类等高访问量信息等
    2. 任务队列，如秒杀、抢购、购票排队等
    3. 即时信息查询，如各位排行榜、各类网站访问统计、公交到站信息、在线人数信息（聊天室、网站）、设备信号等
    4. 时效性信息控制，如验证码控制、投票控制等
    5. 分布式数据共享，如分布式集群架构中的`Session`分离
    6. 消息队列
    7. 分布式锁 `Token`

# Redis基本操作

## 基本操作

1. 退出：`quit、exit`
2. 清屏：`clear`
3. 帮助：`help` 命令名称，`help @组名(如@string)`
4. `help`命令结果参数：
    1. `summary`：描述
    2. `since`：开始版本
    3. `group`：所属群组
5. 授权访问
    1. 指令：`requirepass <password>`
    2. 配置文件：`config set requirepass <password>；config get requirepass`
    3. 主从：`slave`连接`master`三种
        1. 指令：`auth <password>`
        2. 启动参数：`redis-server –a <password>`
        3. 配置文件：`masterauth <password>`

## key通用操作

1. `key  ` 命名规则：表名：主键名：主键值：字段名
2. 删除 `key`：`del key`
3. 判断 `key` 存在：`exists key`
4. 获取 `key` 类型：`type key`
5. 设置 `key` 有效时间：
    1. 时间单位：
        1. 设置秒：`expire key seconds`
        2. 设置毫秒：`pexpire key milliseconds`
    2. `UNIX`时间戳：
        1. 设置秒：`expireat key timestamp`
        2. 设置毫秒：`pexpireat key milliseconds-timestamp`
6. 获取`key`有效时间
    1. `ttl key`
    2. `pttl key`
7. 切换`key`永久：`persist key`
8. 查询`key`：`keys pattern`会阻塞进程
    1. `*`任意个字符如`keys na*`
    2. `?`某个字符如`keys na?m`
9. 重命名`key`
    1. 重名覆盖：`rename key newkey`，目标`key`名已经存在，会拿`key`的值覆盖`newkey`的值，名字变为`newkey`
    2. 重名不覆盖：`renamenx key newkey`，不允许覆盖
10. 对`key`(`List、Set`)内元素排序：`sort key`只能针对`List`或者`Set`

## DB操作(index库索引)

1. 切换数据库：`select index`，`Redis`内置`16`个数据库
2. 测试连接：`ping`
3. 数据移动：`move key index`，当前库存在，目标库不存在才能移动
4. 查看数据库`key`数：`dbsize`
5. 数据清除：
    1. 清当前数据库：`flushdb`
    2. 清除所有数据库：`flushall`

# Redis 常见数据类型的操作

## String操作

1. 添加(覆盖)
    1. 单个：`set key value`
    2. 批量：`mset key1 value1 key2 value2`，实际还是一个一个添加
    3. 设置`key`的生命周期(不存在就创建key)
        1. 秒：`setex key timeout value`
        2. 毫秒：`psetex key timeout value`
    4. 追加：`append key value` ，返回添加之后的长度`num`，不存在该`key`则新建`kv`
2. 获取(不存在返回`null`)
    1. 单个：`get key`
    2. 批量：`mget key1 key2`
    3. 获取字符长度：`strlen key`
3. 删除：`del key`，存在返回1否则0
4. 数值操作
    1. `Redis` 数值范围是 `Long.MAX_VALUE`，越界产生异常，步长 `increment` 可为负
    2. 数值自增：`incr key + 1`、`incrby key increment`、`incrbyfloat key increment`
    3. 数值自减：`decr key - 1`、`decrby key increment`

## List操作(左头右尾)

1. 基本概念
    1. 双向链表(双向队列)，有顺序，可重复数据，最多`2^32 -1`个元素
    2. 具有索引的概念，但通常以栈或者队列方式操作
2. 添加
    1. 头部添加元素：`lpush key value1 [value2]`
    2. 尾部添加元素：`rpush key value1 [value2]`
3. 获取
    1. 头取：`lrange key startindex stopindex`，0首元素索引，-1尾元素索引
    2. 获取指定索引元素：`lindex key index`
    3. 获取`List`的元素个数：`llen key`
4. 删除
    1. 获取并移除元素(**配合添加实现队列和栈**)
        1. 头部移除：`lpop key`
        2. 尾部移除：`rpop key`
    2. 移除指定数据：`lrem key count value`，`count`删除个数，`value`删除的值
    3. 规定时间内获取并删除：存在`key`则删除，否则等待，b相当于`block`
        1. 头取移除：`blpop key1 [key2] timeout`
        2. 尾部移除：`brpop key1 [key2] timeout`
    4. 移动到另一List：`brpoplpush source target timeout`，**源**`List`尾部取插入**目标**`List`头部

## Set操作

1. 基本概念
    1. 与`Hash`存储结构完全相同，仅存储键，不存储值`nil`，不允许重复的值，与`JAVA`的`HashSet`类似
    2. 能够保存大量的数据，高效的内部存储机制，便于查询，解决`List(链表)`查询慢
2. 添加：`sadd key member1 [member2]`
3. 获取
    1. 获取`Set`中所有数据：`smembers key`
    2. 获取`Set`元素个数：`scard key`
    3. 随机获取指定数目元素：`srandmember key [count]`
4. 删除
    1. 批量删除：`srem key member1 [member2]`
    2. 随机获取指定数目元素并删除：`spop key [count]`
    3. 移至另一`Set`：`smove source target member`
5. 判断`Set`是否包含元素：`sismember key member`
6. 扩展操作
    1. 求集合的交、并、差集
        1. 交：`sinter key1 [key2]`
        2. 并：`sunion key1 [key2]`
        3. 差：`sdiff key1 [key2]`
    2. 求集合的交、并、差集并存到新集合中
        1. 交：`sinterstore target key1 [key2]`
        2. 并：`sunionstore destination key1 [key2]`
        3. 差：`sdiffstore destination key1 [key2]`

## Hash操作

1. 基本信息
    1. 每个`Hash`可以存储 `2^32 -1`个键值对，不是为了存储大量对象设计的
    2. 存储结构
        1. 如果`field`数量较少，存储结构优化为类数组结构
        2. 如果`field`数量较多，存储结构使用 `HashMap` 结构
    3. 添加
        1. 单个：`hset key field value`
        2. 批量：`hmset key field1 value1 field2 value2`
        3. 只有字段 `field` 不存在时，设置哈希表字段的值：`hsetnx key field value`
    4. 获取
        1. 单个`field`：`hget key field`
        2. 全部`field`：`hgetall key`
        3. 指定`field`：`hmget key field1 field2`
        4. 删除：`hdel key field1 [field2]`
        5. 获取 `Hash`中字段个数：`hlen key`
        6. 判断是否存在`key ` 的字段：`hexists key field`
        7. 获取 `Hash` 所有 `field`：`hkeys key`
        8. 获取 `Hash` 所有 `field` 值：`hvals key`
    5. 数值操作：
        1. 自增：`hincrby key field increment`
        2. 自增：`hincrbyfloat key field increment`

## ZSet操作

1. 添加：`zadd key score1 member1 [score2 member2]` ，`score`用于排序，重复`member`，覆盖`score`
2. 获取
    1. `score`小到大(获取指定索引)：`zrange key start stop [WITHSCORES]`
    2. `score`大到小(获取指定索引)：`zrevrange key start stop [WITHSCORES]`
    3. 获取数据总量：`zcard key`
    4. 获取`score`范围数据量：`zcount key min max`
    5. 获取元素索引/排名
        1. 小到大：`zrank key member`
        2. 大到小：`zrevrank key member`
    6. score获取/修改
        1. 获取：`zrevrank key member`
        2. 修改：`zincrby key increment member`
3. 删除
    1. 移除指定元素：`zrem key member [member2]`
    2. 条件删除
        1. 根据索引范围删除：`zremrangebyrank key start stop`
        2. 根据`score`范围删除：`zremrangebyscore key min max`
4. 操作(存储到新的`ZSet`里)
    1. 交：`zinterstore target numkeys key [key ...]`
    2. 并：`zunionstore target numkeys key [key ...]`
