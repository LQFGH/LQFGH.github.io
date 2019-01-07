---
layout:     post
title:      "MySql核心技术-mysql变量"
subtitle:   "mysql变量"
date:       2019-01-05 19:00:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql
---

系统变量
* 全局变量
* 会话变量

自定义变量
* 用户变量
* 局部变量


## 系统变量

> 说明：变量由系统提供，不是用户定义

###### 查看所有的系统变量
```sql
SHOW VARIABLES;
```


###### 查看满足条件的系统变量
```sql
SHOW VARIABLES LIKE '%char%';
```


###### 查看指定的系统变量的值
> 变量的名字前面加上 `@@`

```sql
SELECT @@character_set_client;
```


###### 设置环境变量
```sql
SET @@global.autocommit = 0;
```