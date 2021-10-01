---
title: 分布式 ID 之雪花算法实现
date: 2020-03-24 17:23:38
updated:
tags: Java
typora-root-url: ..
---

# 雪花算法介绍

雪花算法（Snowflake）是 Twitter 提出来的一个算法，其目的是生成一个 64 bit 的整数：

![Snowflake](/img/distribute/Snowflake.png)

转换为十进制和十六进制如下。其中十进制的最大长度为 19 位：

![](/img/distribute/max_id.png)

其中：

- 42 bit：用来记录时间戳，表示自 1970-01-01T00:00:00Z 之后经过的毫秒数，其中 1 bit 是符号位。由于精确到毫秒，相比有符号 32 bit 整型所存储的时间戳需要多 10 bit 来记录毫秒数（0-999）。42 bit 可以记录 69 年，如果设置好起始时间比如今年是2018年，那么可以用到 2089 年。42 bit 时间戳范围如下：

    ```Java
    // 0
    long a = 0b00_00000000_00000000_00000000_00000000_00000000L;
    // 999
    long b = 0b00_00000000_00000000_00000000_00000011_11100111L;
    // 2^41-1, 2199023255551
    long c = 0b01_11111111_11111111_11111111_11111111_11111111L;
    
    // 1970-01-01T00:00:00.000Z
    Instant.ofEpochMilli(a).atZone(ZoneOffset.of("-00:00")).toLocalDateTime();
    // 1970-01-01T00:00:00.999Z
    Instant.ofEpochMilli(b).atZone(ZoneOffset.of("-00:00")).toLocalDateTime();
    // 2039-09-07T15:47:35.551Z
    Instant.ofEpochMilli(c).atZone(ZoneOffset.of("-00:00")).toLocalDateTime();
    ```

- 10 bit：用来记录机器 ID，最多可以记录 2^10-1=1023 台机器，一般用前 5 位代表数据中心，后面 5 位是某个数据中心的机器 ID

    ![](/img/distribute/1023.png)

- 12 bit：循环位，用来对同一个毫秒之内产生的 ID，12 位最多可以记录 2^12-1=4095 个，也就是在同一个机器同一毫秒最多记录 4095 个，多余的需要进行等待下个毫秒。

    ![](/img/distribute/4095.png)

上面只是一个将 64 bit 划分的标准，当然也不一定这么做，可以根据不同业务的具体场景来划分，比如下面给出一个业务场景：

- 服务目前 QPS10 万，预计几年之内会发展到百万。
- 当前机器三地部署，上海，北京，深圳都有。
- 当前机器 10 台左右，预计未来会增加至百台。

这个时候我们根据上面的场景可以再次合理的划分 64 bit，QPS几年之内会发展到百万，那么每毫秒就是千级的请求，目前10台机器那么每台机器承担百级的请求（100 W / 1000 ms / 10 台）。为了保证扩展，后面的循环位可以限制到 1024，也就是 2^10，那么循环位 10 位就足够了。

机器三地部署我们可以用 3 bit 总共 8 来表示机房位置；当前机器 10 台，为了保证能够扩展到百台那么可以用 7 bit 总共 128 来表示；时间位依然是 42 bit。那么还剩下 64-10-3-7-42 = 2 bit，剩下 2 bit 可以用来进行扩展。

# 雪花算法实现

下面是我的对雪花算法的实现，涉及几点思考：

1. 为了节省空间、提升运算性能，主要使用到位运算，而不是字符串操作。
2. 为了提高可配置性，将每一部分的位数抽取成了常量，以便自定义。
3. 解决时间回拨问题。如果回拨时长较短（可配置，代码中配了 5 ms），线程等待；如果较长，则利用扩展位避免生成重复 ID（扩展位可配，代码中配置了 3 位，即最多支持 3 次回拨，超出则抛异常）。

```Java
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.time.Instant;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.HashSet;
import java.util.Set;

import static org.junit.Assert.assertEquals;

@Slf4j
public class IdGeneratorTest {

    @Test
    public void test() {
        int initialCapacity = 1000;
        Set<Long> ids = new HashSet<>(initialCapacity);
        IdGenerator idGenerator = new IdGenerator(3, 31);
        
        for (int i = 0; i < initialCapacity; i++) {
            long id = idGenerator.nextId();
            ids.add(id);

            IdGenerator.log(id);
        }
        assertEquals(ids.size(), initialCapacity);
    }

}

@Slf4j
class IdGenerator {

    // 时间戳
    private long timestamp;
    // 数据中心
    private int dataCenterNo;
    // 机器
    private int workerNo;
    // 序列号
    private int seqNo;
    // 扩展位
    private int ext;

    // 数据中心 取值范围（十进制）：0~3
    private static final int DATA_CENTER_NO_BITS = 2;
    // 机器 取值范围（十进制）：0~31
    private static final int WORKER_NO_BITS = 5;
    // 序列号 取值范围（十进制）：0~4095
    private static final int SEQ_NO_BITS = 12;
    // 扩展位 取值范围（十进制）：0~7
    private static final int EXT_BITS = 3;

    // 数据中心 最大值（十进制）：3
    private static final int MAX_DATA_CENTER_NO = (1 << DATA_CENTER_NO_BITS) - 0B1;
    // 机器 最大值（十进制）：31
    private static final int MAX_WORKER_NO = (1 << WORKER_NO_BITS) - 0B1;
    // 序列号 最大值（十进制）：4095
    private static final int MAX_SEQ_NO = (1 << SEQ_NO_BITS) - 0B1;
    // 扩展位 最大值（十进制）：7
    private static final int MAX_EXT = (1 << EXT_BITS) - 0B1;
  
    // 时间戳 左移 22 位
    private static final int TIMESTAMP_SHIFT = DATA_CENTER_NO_BITS + WORKER_NO_BITS + SEQ_NO_BITS + EXT_BITS;
    // 数据中心 左移 20 位
    private static final int DATA_CENTER_NO_SHIFT = WORKER_NO_BITS + SEQ_NO_BITS + EXT_BITS;
    // 机器 左移 15 位
    private static final int WORKER_NO_SHIFT = SEQ_NO_BITS + EXT_BITS;
    // 序列号 左移 3 位
    private static final int SEQ_NO_SHIFT = EXT_BITS;

    // 最大回拨毫秒数
    private static final int MAX_BACKWARD_MILLIS = 5;

    /**
     *
     * @param dataCenterNo 数据中心编号
     * @param workerNo 机器编号
     */
    public IdGenerator(int dataCenterNo, int workerNo) {
        if (Integer.toBinaryString(dataCenterNo).length() > DATA_CENTER_NO_BITS) {
            throw new IllegalArgumentException(String.format("当前数据中心编号 %s 超限，最大支持 %s", dataCenterNo, MAX_DATA_CENTER_NO));
        } else if (Integer.toBinaryString(workerNo).length() > WORKER_NO_BITS) {
            throw new IllegalArgumentException(String.format("当前机器编号 %s 超限，最大支持 %s", workerNo, MAX_WORKER_NO));
        }
        this.dataCenterNo = dataCenterNo;
        this.workerNo = workerNo;
    }

    /**
     * 生成分布式 ID
     * @return 分布式 ID
     */
    public synchronized long nextId() {
        long now = System.currentTimeMillis();
        // init or reset
        if (timestamp == 0 || timestamp < now) {
            timestamp = now;
            seqNo = 0;
        }
        // seqNo increment
        else if (timestamp == now) {
            if (seqNo < MAX_SEQ_NO) {
                seqNo++;
            } else {
                log.warn("序列号已耗尽，等待重新生成。seqNo = {}, MAX_SEQ_NO = {}", seqNo, MAX_SEQ_NO);
                return sleepAndNextId(1);
            }
        }
        // clock backward
        else {
            return nextIdForBackward(now);
        }

        return timestamp << TIMESTAMP_SHIFT
                | dataCenterNo << DATA_CENTER_NO_SHIFT
                | workerNo << WORKER_NO_SHIFT
                | seqNo << SEQ_NO_SHIFT
                | ext;
    }

    private long nextIdForBackward(long now) {
        log.warn("发生时间回拨，timestamp = {}, now = {}", timestamp, now);

        long duration = timestamp - now;
        // 回拨不多，直接等待并重试
        if (duration <= MAX_BACKWARD_MILLIS) {
            return sleepAndNextId(duration);
        }
        // 回拨过多，则使用扩展位
        else {
            if (ext < MAX_EXT) {
                // 将时间戳修正为回拨时间，为防止重复生成 ID，扩展位加一，并重试
                ext++;
                timestamp = now;
                return nextId();
            } else {
                throw new IllegalStateException(String.format("扩展位已耗尽。ext = %s, MAX_EXT = %s", ext, MAX_EXT));
            }
        }
    }

    @SneakyThrows
    private long sleepAndNextId(long millis) {
        Thread.sleep(millis);
        return nextId();
    }

    public static void log(long id) {
        long timestamp = id >> TIMESTAMP_SHIFT;
        long dataCenterNo = id >> DATA_CENTER_NO_SHIFT & MAX_DATA_CENTER_NO;
        long workerNo = id >> WORKER_NO_SHIFT & MAX_WORKER_NO;
        long seqNo = id >> SEQ_NO_SHIFT & MAX_SEQ_NO;
        long ext = id & MAX_EXT;

        log.info("Binary is {}, id is {}", Long.toBinaryString(id), id);
        log.info("Binary is {}, time is {}", Long.toBinaryString(timestamp), getLocalDateTime(timestamp));
        log.info("Binary is {}, dataCenterNo is {}", Long.toBinaryString(dataCenterNo), dataCenterNo);
        log.info("Binary is {}, workerNo is {}", Long.toBinaryString(workerNo), workerNo);
        log.info("Binary is {}, seqNo is {}", Long.toBinaryString(seqNo), seqNo);
        log.info("Binary is {}, ext is {}", Long.toBinaryString(ext), ext);
    }

    private static LocalDateTime getLocalDateTime(long timestamp) {
        return Instant.ofEpochMilli(timestamp).atZone(ZoneId.systemDefault()).toLocalDateTime();
    }

}
```

输出结果：

```
// Binary is 101111000100100011111011111000111010101011111111000000000000000, id is 6783685416349302784
// Binary is 10111100010010001111101111100011101010101, time is 2021-04-02T17:43:58.037
// Binary is 11, dataCenterNo is 3
// Binary is 11111, workerNo is 31
// Binary is 0, seqNo is 0
// Binary is 0, ext is 0
```

二进制分段如下：`10111100010010001111101111100011101010101_11_11111_000000000000_000`

# 延伸阅读：位运算

位运算符如下：

```
&：按位与。
|：按位或。
~：按位非。
^：按位异或。
<<：左位移运算符（M << n = M * 2^n）。
>>：右位移运算符（M >> n = M / 2*n）。
>>>：无符号右移运算符。
```

例如，Java 中求 key 应当放到散列表的哪个位置（offset）：

```Java
static final int getOffset(Object key, int length) 
    int hashcode;
    int hash = (hashcode = key.hashCode()) ^ (hashcode >>> 16);
    int offset = hash & (length - 1);
    log.info("key: {}, hashcode: {}, hash: {}, offset: {}", key, Integer.toBinaryString(hashcode), Integer.toBinaryString(hash), offset);
    return offset;
}
```

参考：

* [《Java 位运算符 &、|、^、~、<<、>>、>>>》](https://www.cnblogs.com/SunArmy/p/9837348.html)
* [《异或的用途》](https://blog.csdn.net/qq_39705793/article/details/81237005)

# UNIX 时间戳

参考：[UNIX 时间戳总结](/2020/02/28/unix-timestamp/)