---
layout:     post
title:      "MyBatis-多参数传递"
subtitle:   "多参数传递"
date:       2019-01-22 02:00
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MyBatis
---

为接口方法传递多个参数，该如何在映射文件中获取参数

```java
	 * 根据id和username获取employee
	 * @param id
	 * @param userName
	 * @return
	 */
	Employee selectByIdAndName(Integer id,String lastName);
```


#### 方法一

使用 `0-N` 获取参数

```xml
	<select id="selectByIdAndName" resultType="com.mybatis.bean.Employee">
		select * from tbl_employee where id = #{0} and last_name = #{1}
	</select>
```


#### 方法二

使用 `param1-paramN`获取参数

```xml
	<select id="selectByIdAndName" resultType="com.mybatis.bean.Employee">
		select * from tbl_employee where id = #{param1} and last_name = #{param2}
	</select>
```
