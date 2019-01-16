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