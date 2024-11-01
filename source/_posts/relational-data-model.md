---
title: 关系数据模型总结
date: 2020-04-25 21:48:49
updated:
tags: MySQL
typora-root-url: ..
---

# 数据特征

数据具有两种特征：

> 静态特征，指的是数据结构、数据间的联系、对数据取值范围的约束。
>
> 动态特征，指的是对数据可以进行符合一定规则的操作

# 数据模型

## 组成要素

![composition_of_data_model](/img/db/data-model/composition_of_data_model.png)

## 模型分类

![classification_of_data_model](/img/db/data-model/classification_of_data_model.png)

# 关系数据模型

## 关系数据结构

![data_structure_of_relational_data_model](/img/db/data-model/data_structure_of_relational_data_model.png)

## 关系的完整性约束

![data_check_of_relational_data_model](/img/db/data-model/data_check_of_relational_data_model.png)

## 关系数据语言

关系语言是一种声明式的查询语言，基于声明式编程范式（Declarative），有别于命令式编程范式（Imperative），特点（优点）是：**高度非过程化**！

![relational_language_of_relational_data_model](/img/db/data-model/relational_language_of_relational_data_model.png)

# 关系数据库的规范化理论

![normalized_form](/img/db/data-model/normalized_form.png)

![normalized_form](/img/db/data-model/normalized_form.jpg)

![normalized_form](/img/db/data-model/normalized_form_examples.jpg)

# 参考

[数学符号表（维基百科）](https://zh.wikipedia.org/wiki/%E6%95%B0%E5%AD%A6%E7%AC%A6%E5%8F%B7%E8%A1%A8)

[关系代数（维基百科）](https://zh.wikipedia.org/wiki/%E5%85%B3%E7%B3%BB%E4%BB%A3%E6%95%B0_(%E6%95%B0%E6%8D%AE%E5%BA%93))

[关系代数（百度百度）](https://baike.baidu.com/item/%E5%85%B3%E7%B3%BB%E4%BB%A3%E6%95%B0)

[SQL 能完成哪方面的计算？一文详解关系代数和 SQL 语法 | 阿里技术](https://mp.weixin.qq.com/s/D8Rv-E_gSYFhnscVMK1WGg)