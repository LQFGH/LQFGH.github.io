---
layout:     post
title:      "MySql高级技术-show profile"
subtitle:   "show profile"
date:       2019-01-16 20:00:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql高级
---




#### **查看 `show profile` 是否已经开启**

```sql
SHOW VARIABLES LIKE 'profiling'
```

![查看show profile是否开启](/img/in-post/mysql-show-profile.jpg)


#### **设置 `show profile` 开启**

```sql
SET profiling=ON
```


#### **执行sql语句**

> 这里执行SQL语句只是为了看 `show profile` 能不能记录SQL执行的情况,结果不是重点。
> 这里主要注意的是最后执行的时间，因为这里执行的时间在 `show profile` 中会被统计出来。

###### 第一条

```sql
SELECT * FROM dept;
```

![第一条sql执行结果](/img/in-post/mysql-show-profile1.jpg)

###### 第二条

> 我这里的 `emp` 表中有50万条记录

```sql
SELECT * FROM emp GROUP BY id/10 LIMIT 200000;
```

![第二条sql执行结果](/img/in-post/mysql-show-profile2.jpg)


###### 第三条

```sql
select * from emp left outer join dept on emp.`deptno` = dept.`deptno`;
```

![第三条sql结果](/img/in-post/mysql-show-profile3.jpg)

###### 第四条

```sql
SELECT * FROM dept LEFT OUTER JOIN emp ON emp.`deptno` = dept.`deptno`;
```

![第四条sql结果](/img/in-post/mysql-show-profile4.jpg)


#### 查看 `show profile` 记录

```sql
show profiles;
```


![查看 `show profile`](/img/in-post/mysql-show-profile5.jpg)

> `Duration` 是执行时间

#### 使用 `show profile` 参数查看详情

###### 一、具体参数

![`show profile` 参数](/img/in-post/mysql-show-profile7.jpg)

###### 二、使用

```sql
show profile cpu,block io for query 3;
show profile cpu,block io for query 4;
```

![](/img/in-post/mysql-show-profile6.jpg)

> 这里发现`Sending data`耗费的时间格外的多，就将查询到的`Sending data`的解释贴在这里是`Sending data`

> `Sending data` 这个状态的名称很具有误导性，所谓的`Sending data`并不是单纯的发送数据，而是包括“收集 + 发送 数据”。
> 这里的关键是为什么要收集数据，原因在于：`mysql`使用“索引”完成查询结束后，`mysql`得到了一堆的行`id`，如果有的列并不在索> > 引中，`mysql`需要重新到“数据行”上将需要返回的数据读取出来返回个客户端。

###### 三、 这里将查询的某些参数去掉在执行查询

> 这里将之前的 `*` 换成了具体的几个参数
> 发现 `Sending data` 比之前少了很多

```sql
select empno,dept.deptno,dname from emp left outer join dept on emp.`deptno` = dept.`deptno`;
```

![](/img/in-post/mysql-show-profile8.jpg.jpg)


###### 四、还有哪些需要格外注意的项

![需要注意的出现的项](/img/in-post/mysql-show-profile7.jpg)


#### 最后补充一个知识点，是可以记录所有执行过的sql的信息-全局查询日志

> 这里只推荐在测试环境使用

###### 开启全局查询日志

```sql
SET GLOBAL general_log=1; --设置开启全局查询日志
SET GLOBAL log_output='TABLE'; --设置将查询日志输出到表中，mysql.general_log表
```

###### 查看全局查询日志

> 执行几条sql之后，查看全局查询日志

```sql
select * from mysql.general_log;
```
![](/img/in-post/mysql-show-profile10.jpg)