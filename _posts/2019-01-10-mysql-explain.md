---
layout:     post
title:      "MySql高级技术-explain"
subtitle:   "explain"
date:       2019-01-05 23:00:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql高级
---


## 作用

使用EXPLAIN关键字可以模拟优化器执行sql查询语句，从而知道mysql是怎么执行查询语句。分析查询语句或者是表结构的性能瓶颈


## 能干什么

* 表的读取顺序
* 数据读取操作的操作类型
* 哪些索引可以使用
* 哪些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询


## 怎么用

```sql
explain select * from employees where employee_id = 195;
```
结果
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table     | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | employees | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+-------+