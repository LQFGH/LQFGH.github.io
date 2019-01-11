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

***



使用EXPLAIN关键字可以模拟优化器执行sql查询语句，从而知道mysql是怎么执行查询语句。分析查询语句或者是表结构的性能瓶颈


## 能干什么

***



* 表的读取顺序
* 数据读取操作的操作类型
* 哪些索引可以使用
* 哪些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询


## 怎么用

***



```sql
EXPLAIN
SELECT * 
FROM employees e
JOIN departments d
ON e.department_id = d.department_id
JOIN locations l
ON d.location_id = l.location_id
```

**结果**

![执行计划结果截图](/img/in-post/mysql-explain.jpg)


## 结果分析

***

#### **id**

* id相同的情况：按照table列的顺序执行查询
* 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

 > 主要反应表的加载顺出
 
 
####  **select_type**

- [ SIMPLE] 简单的select查询，查询中不包含子查询或者UNION
- [ PRIMARY] 查询中包含任何复杂的子部分，则最外层查询被标识为primary
- [ SUBQUERY] 在select或where中使用子查询
- [ DERIVED] 在from列表中包含的子查询被标记DERIVED（衍生），mysql会递归执行这些子查询，把结果放到临时表中
- [ UNION] 若第二个select出现在union之后，则被表级为union；若特包含在from子句的子查询中，外层select将被标记为：DERIVED
- [ UNION RESULT] 从union表获取结果的select

> 说明是什么类型的查询


#### **type**

***



- [ system] 表只有一条记录，不用考虑
- [ const] 表示通过索引一次就找到了，const用于比较primary key或union索引
- [ eq_ref] 唯一索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
- [ ref] 非唯一性索引扫描，返回匹配某个单独值的所有行。根据索引找出集合
- [ range] 值检索给定的范围的行，使用一个索引来选择行。key列显示使用了那个索引，一般就是你的where语句中出现了between、>、<、in、等查询
- [ index] 索引扫描，要比ALL快
- [ ALL] 扫描全表  

访问类型从最好到最差依此是：
	system>const>eq_ref_ref>range>index>ALL


#### **possible_keys和key**

***



possible_keys:可能用到的索引
key:1、实际用到的索引，如果为NULL，则没有使用到索引。
	2、查询中若使用了覆盖索引，则该索引仅出现在key列表中。


#### **key_len**

***

* 表示索引中使用的字节数，可用过该列计算查询中使用的索引的长度。在不损失精度的情况下，长度越短越好。
* key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义


#### **ref**

***

显示索引的那一列被使用了，如果可能的话，是一个常数


#### **rows**

***
查看每张表有多少条被优化


#### **Extra**

***

> 前两个是对性能有负面影响的

包含不适合在其他列中存放的重要信息

![extra对比](/img/in-post/mysql-explain1.jpg)


* using filesort：说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序完成的排序操作成为‘文件排序’
* using temporary：使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by
* useing index：表示相应的select操作中使用了覆盖索引（coverint index），避免访问了表的数据行，效率不错！；如果同事出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查询动作。
* using where：使用了where过滤
* using join buffer：使用了连接缓存
* impossible where：使用了不可能成立的查询条件
* select tables optimezed away
* distinct