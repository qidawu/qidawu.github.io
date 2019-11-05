---
title: MySQL 几种排序机制分析及优化
date: 2019-01-03 15:22:13
updated:
tags: MySQL
typora-root-url: ..
---

# 排序流程

四种排序情况的流程（参考《极客时间》专栏）：

![order_by_process](/img/mysql/order_by_process.png)

![order_by_process](/img/mysql/order_by_process_2.png)

# 总结

排序总结：

![order_by](/img/mysql/order_by.png)

外部排序总结：

* MySQL 会给每个线程分配一块内存用于排序，称为 `sort_buffer`。对 `sort_buffer` 中的数据按照排序字段做**快速排序**；
* 排序可能在内存中完成，也可能需要使用磁盘排序，取决于排序所需的内存和参数 `sort_buffer_size`。
* 内存放不下时，使用外部磁盘排序，外部磁盘排序一般使用**归并排序**算法。可以简单理解，MySQL 将需要排序的数据分成 N 份，每一份单独排序后存在这些临时文件中。然后把这 N 个有序文件再合并成一个有序的大文件。

优化方式：

* 将 `WHERE` 和 `ORDER BY` 子句用到的字段，添加联合索引（注意字段顺序）；
* 如果 `GROUP BY` 结果无需排序，可以加上 `ORDER BY NULL`。

# 参考

https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html
