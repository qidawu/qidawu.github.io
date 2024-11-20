---
title: Redis 实战系列（一）常用数据结构及使用场景总结
date: 2019-06-05 17:13:35
updated:
tags: [Redis, 数据结构]
typora-root-url: ..
---

# 数据结构

## Strings

|                | 单个                                                         | 批量                                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 设值           | `SET` key value [NX &#124; XX] [GET] [EX seconds &#124; PX milliseconds &#124; EXAT unix-time-seconds &#124; PXAT unix-time-milliseconds &#124; KEEPTTL]<br/>~~`SETNX` key value<br/>`SETEX` key seconds value<br/>`PSETEX` key milliseconds value<br/>`GETSET` key value<br/>~~`SETRANGE` key offset value | `MSET` key value [key value ...]<br/>`MSETNX` key value [key value ...] |
| 取值           | `GET` key<br/>`GETDEL` key<br/>`GETRANGE` key start end<br/>`STRLEN` key | `MGET` key [key ...]                                         |
| 原子递增、递减 | `INCR` key<br/>`INCRBY` key increment<br/>`INCRBYFLOAT` key increment<br/>`DECR` key<br/>`DECRBY` key decrement |                                                              |
| 追加           | `APPEND` key value                                           |                                                              |
| 位操作         | `SETBIT` key offset value<br/>`GETBIT` key offset<br/>`BITCOUNT` key [start end]<br/>`BITOP` operation destkey key [key ...]<br/>`BITFIELD` ...<br/>`BITPOS` key bit [start] [end] |                                                              |

使用场景：

* WEB 集群下的 Session 共享：

    ```bash
    $ SET key value
    ```
    
* 分布式锁：

    ```bash
    -- 返回 1 表示加锁成功，0 表示加锁失败
    $ SET key value NX
    $ SETNX key value

    -- 解锁
    $ DEL key
    ```

* 全局计数器：

  ```bash
  $ INCR key
  $ DECR key
  ```

* 分布式流水号：

    ```bash
    $ INCR key
    $ INCRBY key
    ```
    

分布式流水号 Java 伪代码如下，单机一次性取 1000 个 ID，以降低网络开销和 Redis 负载：

```Java
private int id;
private int maxId;
private int INCR_BY = 1000;

public synchronized int nextId() {
  if (id == 0 || id == maxId) {
    maxId = eval(incrby id INCR_BY);
    id = maxId - INCR_BY + 1;
    return id;
  } else {
    return id++;
  }
}
```

## Hashes

散列表，一种通过**散列函数**计算对应数组下标，并通过下标随机访问数据时，时间复杂度为 `O(1)` 的特性快速定位数据的数据结构。Redis 散列表使用这种数据结构来快速获取指定 field。

|                           | 单个                                                         | 批量                                          |
| ------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| 设置 field 的 value       | `HSET` key field value [field value ...]<br/>`HSETNX` key field value | ~~`HMSET` key field value [field value ...]~~ |
| 获取 field 的 value       | `HGET` key field<br/>`HSCAN` key cursor [MATCH pattern] [COUNT count]<br/>`HSTRLEN` key field | `HMGET` key field [field ...]                 |
| 获取所有 fields           | `HKEYS` key                                                  |                                               |
| 获取所有 values           | `HVALS` key                                                  |                                               |
| 获取所有 fields 和 values | `HGETALL` key                                                |                                               |
| 获取 field 的个数         | `HLEN` key                                                   |                                               |
| 判断 field 是否存在       | `HEXISTS` key field                                          |                                               |
| 删除 field                | `HDEL` key field [field ...]                                 |                                               |
| 原子递增、递减指定 field  | `HINCRBY` key field increment<br/>`HINCRBYFLOAT` key field increment |                                               |

## Lists

双端队列。

![redis_lists](/img/cache/redis/redis_lists.png)

| 队列操作           | 左                                                           | 右                                                           |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 入队               | `LPUSH` key element [element ...]<br/>`LPUSHX` key element [element ...] | `RPUSH` key element [element ...]<br/>`RPUSHX` key element [element ...] |
| 出队               | `LPOP` key                                                   | `RPOP` key                                                   |
| 阻塞出队           | `BLPOP` key [key ...] timeout                                | `BRPOP` key [key ...] timeout                                |
| 插队               | `LINSERT` key BEFORE&#124;AFTER pivot element                |                                                              |
| 获取指定索引的元素 | `LINDEX` key index                                           |                                                              |
| 获取指定范围的元素 | `LRANGE` key start stop                                      |                                                              |
| 获取列表长度       | `LLEN` key                                                   |                                                              |
| 覆盖元素           | `LSET` key index element                                     |                                                              |
| 移除元素           | `LREM` key count element                                     |                                                              |
| 移除指定范围的元素 | `LTRIM` key start stop                                       |                                                              |

|                          | 非阻塞                         | 阻塞                                    |
| ------------------------ | ------------------------------ | --------------------------------------- |
| 出队并重新入队另一个队列 | `RPOPLPUSH` source destination | `BRPOPLPUSH` source destination timeout |

使用场景：

* Stack (FILO): `LPUSH` + `LPOP`
* Queue (FIFO): `LPUSH` + `RPOP`，实现简单的消息队列
* Unbounded Blocking Queue (FIFO): `LPUSH` + `BRPOP`

## Sets

无序集合（散列表实现）。

集合操作：

| 集合操作                           | 命令                                             |
| ---------------------------------- | ------------------------------------------------ |
| 添加元素（去重）                   | `SADD` key member [member ...]                   |
| 移除元素                           | `SREM` key member [member ...]                   |
| 判断指定元素是否存在               | `SISMEMBER` key member                           |
| 获取元素个数                       | `SCARD` key                                      |
| 增量式遍历集合元素                 | `SSCAN` key cursor [MATCH pattern] [COUNT count] |
| 获取所有元素                       | `SMEMBERS` key                                   |
| 获取指定个数的**随机元素**         | `SRANDMEMBER` key [count]                        |
| 移除指定个数的**随机元素**，并返回 | `SPOP` key [count]                               |

集合运算：

| 集合运算                           | 数学符号 | 命令                                    |
| ---------------------------------- | -------- | --------------------------------------- |
| 移动指定元素到另一个集合           |          | `SMOVE` source destination member       |
| 求交集（Intersection）             | ∩        | `SINTER` key [key ...]                  |
| 求交集（Intersection），并保存结果 | ∩        | `SINTERSTORE` destination key [key ...] |
| 求并集（Union）                    | ∪        | `SUNION` key [key ...]                  |
| 求并集（Union），并保存结果        | ∪        | `SUNIONSTORE` destination key [key ...] |
| 求差集（Difference）               | −        | `SDIFF` key [key ...]                   |
| 求差集（Difference），并保存结果   | −        | `SDIFFSTORE` destination key [key ...]  |

使用场景：

* 抢红包、抽奖、秒杀 —— 本质上都是同一类问题，解决思路类似。为了减少对临界资源的竞争，避免使用各种锁进行并发控制，可以预先对临界资源进行拆分，以提升性能：

  ```bash
  # 预拆红包，放入集合
  $ SADD key 子红包ID1 [子红包ID2 …]
  # 查看所有红包
  $ SMEMBERS key
  # 随机抽取红包
  $ SPOP key [count]
  
  # 预先将奖品放入奖池
  $ SADD key member [member …]
  # 查看所有奖品
  $ SMEMBERS key
  # 随机抽奖（只抽一次）
  $ SRANDMEMBER key [count]
  
  # 登记参与抽奖的候选人
  $ SADD key member [member …]
  # 查看所有候选人
  $ SMEMBERS key
  # 随机抽取一二三等奖的获得者
  $ SPOP key [count]
  ```

* 社交应用的关注模型：

  ```bash
  # 我关注的人
  $ SMEMBERS key
  
  # 求共同关注
  $ SINTER key [key ...]
  
  # 我关注的人也关注 ta
  foreach(member in 我_关注的人) {
    # 我每个关注的人，他们关注的人中，是否有 ta
    $ SISMEMBER member_关注的人 ta
  }
  
  # 我可能认识的人
  foreach(member in 我_关注的人) {
    # 我每个关注的人，他们关注的人中，有我还没关注的
    $ SDIFF member_关注的人 我_关注的人
  }
  ```

* 商品筛选

  ```bash
  # 1、分类筛选维度，每个维度都为一个集合
  # 2、将商品按维度加入所属集合
  # 3、多选筛选条件时，求交集
  $ SINTER key [key ...]
  
  # 4、根据交集 member，获取商品详情（O(1) 时间复杂度）
  foreach member {
    # 每个 field 为商品属性
    $ HGETALL member
  }
  ```

## Sorted Sets

有序集合（复合数据结构实现：散列表+跳表）。

使用场景：

* Top K（例如排行榜）。实现思路：利用集合的三大特性之一——互异性，进行去重，相同元素只进行计数，形成一个二元组集合（key 为元素，value 为计数）。最后按计数结果对集合进行倒序排序，取前 N 个元素。

# 其它命令

|                           | 命令     |
| ------------------------- | -------- |
| 删除 key                  | `DEL`    |
| 设值 key 的过期时间（秒） | `EXPIRE` |

# 参考

https://redis.io/commands