---
layout:     post
title:      "MySql高级技术-慢查询日志"
subtitle:   "慢查询日志"
date:       2019-01-14 19:00:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql高级
---

* MySQL的提供的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中相应时间超过阙值的语句，具体只运行时间超过 `long_query_time` 只的sql，则会被记录在慢查询日志中
* 具体运行时间超过 `long_query_time` 值的sql,则会被记录到慢查询日志中。 `long_query_time `的默认值为10秒，意思是执行十秒钟以上的sql被记录到慢查询日志中。
  * 由慢查询日志查看哪些sql执行时间超过 `long_query_time`  

#### **慢查询日志使用**


###### 查看是否开启了慢查询日志
```sql
SHOW VARIABLES LIKE '%slow_query_%'
```

> `slow_query_log_file` 的值为慢查询日志的存放路径

![](img/in-post/mysql-slow-log.jpg)

###### 开启慢查询日志
```sql
SET GLOBAL slow_query_log = 1;
```

> 当设置完参数立马查询发现参数还是off，需要关闭控制台重新打开再查询
> 还可以在配置文件中设置参数，但是不推荐在配置文件中设置

###### 查看慢查询阙值
```sql
SHOW VARIABLES LIKE '%long_query%'
```

###### 设置阙值
```sql
SET GLOBAL long_query_time = 3;
```

###### 测试慢查询
```sql
select sleep(4);
SELECT SLEEP(5)
```

> 查看上面查出来的 `slow_query_log_file` 日志文件如图所示,超过阙值时间的被记录在了慢查询日志中

![](img/in-post/mysql-slow-query1.jpg)


###### 查看慢查询日志数量
```sql
show global status like '%Slow_queries%';
```



#### **日志分析工具mysqldumpslow**

**mysqldumpslow的参数**
![](img/in-post/mysql-slow-log2.jpg)

**常用用法**
![](img/in-post/mysql-slow-log3.jpg)