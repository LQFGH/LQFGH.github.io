---
layout:     post
title:      "MyBatis-获取生成主键"
subtitle:   "获取生成主键"
date:       2019-01-22 02:00
author:     "LQFGH"
header-img: "img/mybatis-logo.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MyBatis
---


img/mybatis-logo.jpg

#### `MySQL` 保存并获取主键

```xml
	<insert id="addEmp" keyProperty="id" useGeneratedKeys="true" databaseId="mysql">
		insert into tbl_employee(last_name,gender,email) values(#{lastName},#{gender},#{email})
	</insert>
```


#### `Oracle` 保存并获取主键

> 先获取主键再插入记录

```xml
	<insert id="addEmp" databaseId="oracle" >
		<selectKey keyProperty="id" resultType="Integer" order="BEFORE">
			select emp_seq.nextval from dual
		</selectKey>
		insert into emp(id,last_name,email) values(#{id},#{lastName},#{email})
	</insert>
```


> 先插入记录再获取主键

```xml
	<insert id="addEmp" databaseId="oracle" >
		<selectKey keyProperty="id" resultType="Integer" order="AFTER">
			select emp_seq.currval from dual
		</selectKey>
		insert into emp(id,last_name,email) values(emp_seq.nextval,#{lastName},#{email})
	</insert>
```