---
layout:     post
title:      "MySql高级技术-解决mysql不能远程连接"
subtitle:   "Host xxx is not allowed to connect to this MySQL server 问题"
date:       2019-01-20 01:30:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql高级
---

文章参考： [](https://blog.csdn.net/qq_39781497/article/details/81302950)

本地登陆远程MySQL是报错 `Host xxx is not allowed to connect to this MySQL server` 

###### 登陆远程主机看下mysql库中的user表中的 `root` 用户的 `host` 地址

```sql
use mysql;
select host from user where user='root';
```


###### 将host地址改为 '%' 即可

```sql
update user set host = '%' where user = 'root';
```

###### 设置完了刷新下权限相关表

```sql
FLUSH PRIVILEGES;
```