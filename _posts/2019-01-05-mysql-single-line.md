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

## 单行函数

***






## 表的修改

***



> 表的修改一般都是
> ALTER TABLE XXX ADD|DROP|MODIFY|CHANGE COLUMN 列名 [列类型，约束]

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


