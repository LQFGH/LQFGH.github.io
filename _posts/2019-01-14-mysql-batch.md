---
layout:     post
title:      "MySql高级技术-批量插入脚本编写"
subtitle:   "批量插入脚本编写"
date:       2019-01-14 23:00:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql高级
---

#### 创建数据库并使用
```sql
create database bigData;
use bigData;
```

#### 创建dept表
```sql
-- 创建dept
CREATE TABLE dept(
	id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
	deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
	dname VARCHAR(20) NOT NULL DEFAULT '',
	loc VARCHAR(13) NOT NULL DEFAULT ''
)ENGINE=INNODB DEFAULT CHARSET=GBK;
```

#### 创建emp表
```sql
-- 创建emp
CREATE TABLE emp(
	id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
	empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
	ename VARCHAR(20) NOT NULL DEFAULT '',
	job VARCHAR(9) NOT NULL DEFAULT '',
	mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
	hiredate DATE NOT NULL,
	sal DECIMAL(7,2) NOT NULL,
	comm DECIMAL(7,2) NOT NULL,
	deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0
)ENGINE=INNODB DEFAULT CHARSET=GBK;
```

#### 修改全局参数，防止批量插入是报错
```sql
-- 查看参数
show variables like 'log_bin_trust_function_creators';
-- 修改全局参数，防止批量插入是报错
set global log_bin_trust_function_creators = 1;
```

#### 编写随机字符串函数
```sql
delimiter $$
create function  rand_string(n int) returns varchar(255)
begin
	declare chars_str varchar(100) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
	declare return_str varchar(255) default '';
	declare i int default 0;
	while i < n do
	set return_str = concat(return_str,substring(chars_str,floor(1+rand()*52),1));
	set i = i + 1;
	end while;
	return return_str;
	return return_str;
end $$

```

编写随机整数函数
```sql
delimiter $$
create function rand_num() returns int(5)
begin
	declare i int default 0;
	set i = floor(100+rand()*10);
	return i;
end $$

```

#### 创建插入员工存储过程
```sql
delimiter $$
create procedure insert_emp(in start int(10),in max_num int(10))
begin
	declare i int default 0;
	set autocommit = 0;
	repeat
	set i = i + 1;
	insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values((start+i),rand_string(6),'SALESMAN',0001,curdate(),2000,400,rand_num());
	until i = max_num
	end repeat;
	commit;
end $$

```

#### 创建插入部门存储过程
```sql
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0;
	REPEAT
	SET i = i + 1;
	INSERT INTO dept(deptno,dname,loc) values((start+i),rand_string(10),rand_string(8));
	UNTIL i = max_num
	END REPEAT;
	COMMIT;
END $$

```

#### 调用插入部门和员工的存储过程
```sql
call insert_dept(100,10);
call insert_emp(100001,500000);
```