---
title: 数据结构系列（一）脑图总结
date: 2019-06-02 17:13:35
updated:
tags: Redis
typora-root-url: ..
---

# 引言

![计算机解决问题的步骤](/img/data-structure/problem_solution_steps.png)

# 基本概念和术语

![data_structure](/img/data-structure/data_structure_terms.png)

* 数据
* 数据对象：是**性质相同**的数据元素的集合，是数据的子集。
* 数据元素：数据的**基本单位**，也是运算的基本单位。例如在数据库中，又称为记录（Record）/元组（Tuple）。
* 数据项：数据不可分割的**最小单位**。例如在数据库中，又称为字段（Field）/域（Domain）。
* 数据结构：是相互之间存在一种或多种特定关系的数据元素的集合。它包括：
  * 数据的逻辑结构
  * 数据的存储结构
  * 数据的基本运算

## 逻辑结构

|              | 线性结构                                                     | 树形结构                             | 图结构 |
| ------------ | ------------------------------------------------------------ | ------------------------------------ | ------ |
| 顺序存储方式 | 数组                                                         | 数组<br/>* 完全二叉树<br/>* 满二叉树 | -      |
| 链式存储方式 | 链表<br/>* 单向链表<br/>* 双向链表<br/>* 单向循环链表<br/>* 双向循环链表 | 链表                                 | 链表   |

### 离散集合

### 线性结构

### 树形结构

### 图结构

## 存储结构

数据的逻辑结构在计算机中的实现，成为数据的存储结构（或物理结构）。

一般情况下，一个存储结构包括以下两个部分：

* 存储数据元素本身
* 数据元素之间的关联方式

### 顺序存储方式

### 链式存储方式

### 索引存储方式

### 散列存储方式

# 常用数据结构

![data_structure](/img/data-structure/data_structure.png)

# 参考

https://www.cs.usfca.edu/~galles/visualization/Algorithms.html

https://www.bigocheatsheet.com

[逻辑结构与物理结构](https://baozoulin.gitbook.io/-data-structure/chapter1/12-luo-ji-jie-gou-yu-wu-li-jie-gou)