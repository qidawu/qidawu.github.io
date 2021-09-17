---
title: Redis 实战系列（一）常用数据结构及使用场景总结
date: 2019-06-05 17:13:35
updated:
tags: Redis
typora-root-url: ..
---

# 数据结构

## Strings

|                | 单个                                                         | 批量                                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 获取           | `GET` key<br/>`STRLEN` key<br/>`GETRANGE` key start end      | `MGET` key [key ...]                                         |
| 设值           | `SET` key value [EX seconds&#124;PX milliseconds] [NX&#124;XX] [KEEPTTL]<br/>`SETNX` key value<br/>`SETEX` key seconds value<br/>`PSETEX` key milliseconds value<br/>`GETSET` key value<br/>`SETRANGE` key offset value | `MSET` key value [key value ...]<br/>`MSETNX` key value [key value ...] |
| 原子递增、递减 | `INCR` key<br/>`INCRBY` key increment<br/>`INCRBYFLOAT` key increment<br/>`DECR` key<br/>`DECRBY` key decrement |                                                              |
| 追加           | `APPEND`                                                     |                                                              |
| 位操作         | `SETBIT` key offset value<br/>`GETBIT` key offset<br/>`BITCOUNT` key [start end]<br/>`BITOP` operation destkey key [key ...]<br/>`BITFIELD` ...<br/>`BITPOS` key bit [start] [end] |                                                              |

使用场景：

* WEB 集群下的 Session 共享：

    ```bash
    $ SET key value
    ```
    
* 分布式锁：

    ```bash
    -- 返回 1 表示加锁成功，0 表示加锁失败
    $ SETNX key value
    $ SET key value NX

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
| 获取                      | `HGET` key field<br/>`HSCAN` key cursor [MATCH pattern] [COUNT count]<br/>`HSTRLEN` key field | `HMGET` key field [field ...]                 |
| 获取所有 fields           | `HKEYS` key                                                  |                                               |
| 获取所有 values           | `HVALS` key                                                  |                                               |
| 获取所有 fields 和 values | `HGETALL` key                                                |                                               |
| 判断 field 是否存在       | `HEXISTS` key field                                          |                                               |
| 获取 field 个数           | `HLEN` key                                                   |                                               |
| 设值                      | `HSET` key field value [field value ...]<br/>`HSETNX` key field value | ~~`HMSET` key field value [field value ...]~~ |
| 原子递增、递减            | `HINCRBY` key field increment<br/>`HINCRBYFLOAT` key field increment |                                               |
| 删除 fields               | `HDEL` key field [field ...]                                 |                                               |

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

无序集合。

集合操作：

| 集合操作                           | 命令                                             |
| ---------------------------------- | ------------------------------------------------ |
| 添加元素                           | `SADD` key member [member ...]                   |
| 移除元素                           | `SREM` key member [member ...]                   |
| 判断指定元素是否存在               | `SISMEMBER` key member                           |
| 获取所有元素                       | `SMEMBERS` key                                   |
| 获取元素个数                       | `SCARD` key                                      |
| 增量式遍历集合元素                 | `SSCAN` key cursor [MATCH pattern] [COUNT count] |
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

* 去重

  ```bash
  $ SADD key member [member ...]
  ```
  
* 抽奖：

  ```bash
  # 随机抽奖（只抽一次）
  $ SRANDMEMBER key [count]
  
  # 随机抽取一二三等奖
  $ SPOP key [count]
  ```

* 社交应用的关注模型：

  ```bash
  # 我关注的人
  $ SMEMBERS key
  
  # 求共同关注
  $ SINTER key [key ...]
  
  # 我关注的人也关注 ta
  foreach(member in 我关注的人) {
    # 我每个关注的人，他们关注的人中，是否有 ta
    $ SISMEMBER member_关注的人 ta
  }
  
  # 我可能认识的人
  foreach(member in 我关注的人) {
    # 我每个关注的人，他们关注的人中，有我还没关注的
    $ SDIFF member_关注的人 我关注的人
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

有序集合。

使用场景：

* Top K（例如排行榜）。实现思路：利用集合的三大特性之一——互异性，进行去重，相同元素只进行计数。最后按计数结果对集合进行倒序排序，取前 N 个元素。

# 其它命令

|                           | 命令     |
| ------------------------- | -------- |
| 删除 key                  | `DEL`    |
| 设值 key 的过期时间（秒） | `EXPIRE` |

# 参考

https://redis.io/commands