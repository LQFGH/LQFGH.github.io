---
layout:     post
title:      "MyBatis-获取生成主键"
subtitle:   "获取生成主键"
date:       2019-01-22 02:00
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MyBatis
---


`MyBatis` 获取主键分为获取自增主键和非自增主键，这里用 `MySQL` 代表自增主键，用 `Oracle` 代表非自增主键

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