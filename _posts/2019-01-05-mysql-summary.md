---
layout:     post
title:      "MySql核心技术"
subtitle:   "mysql核心知识点总结"
date:       2019-01-05 19:00:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql
---

## 查看命令

***

> 通常为 SHOW DATABASES|TABLES|COLUMN 
> 查看列时需要加上表名 SHOW COLUMN FROM table_name

###### 查看数据库有哪些
```sql
SHOW DATABASES
```


###### 查看数据库中表有哪些
> 前提是已经在某一个数据库中，使用命令 `USE database_name`
```sql
SHOW TABLES;
```


###### 查看表中有哪些行
```sql
SHOW COLUMN FROM table_name
```



## 单行函数

***






## 表的修改

***



> 表的修改一般都是
> `ALTER TABLE XXX ADD|DROP|MODIFY|CHANGE COLUMN` 列名 [列类型，约束]

###### 重命名表
```sql
rename table empp to emp;
```


###### 添加字段
```sql
ALTER TABLE emp
ADD(email VARCHAR(20) UNIQUE COMMENT '邮箱')
```


###### 删除字段
```sql
ALTER TABLE emp
DROP COLUMN email;
```


###### 修改字段
```sql
alter table emp
modify column department_id varchar(20);
```


###### 字段重命名
```sql
alter table emp
change column last_name `name` varchar(20);
```



## 数据类型

***


###### 整形

|名称\类型|  tinyint   |  smallint   |  mediumint   |  int\integer   |  bigint   |
|---| --- | --- | --- | --- | --- |
|  字符数 |  1   |  2   |  3   |  4   |  8    |

> 整形字段有无符号通过 `unsigned` 来设置


###### 浮点型




