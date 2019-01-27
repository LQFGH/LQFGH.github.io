---
layout:     post
title:      "MyBatis-原理"
subtitle:   "sqlSessionFactory初始化"
date:       2019-01-27 11:10
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MyBatis
  - Source Code
---

>  总结：
>  1、整个 `sqlSessionFactory` 的过程就是把 配置文件的信息解析并存放到 `Configuration` 中，并创建 			  	            `DefaultSqlSessionFactory` 实例返回给 `sqlSessionFactory`
> 
> 2、`configuration` 贯穿整个初始化的流程，保存了所有配置文件的详细信息



  **`sqlSessionFactory` 初始化过程时序图**
  
  ![](/img/in-post/mybatis-sqlsessionfactory5.jpg)


* Configuration:保存configuration配置文件中素有的配置文件
* XMLConfigBuilder：解析Configuration配置文件
* XMLMapperBuilder：解析Mapper配置文件
* XMLStatementBuilder：解析增删改查标签
* XPathParser：具体解析节点信息的类

***



#### 简单的运行代码

> 由于是看源码，这里的只有两个查询方法和非常简单的配置


###### 全局配置文件

![](/img/in-post/mybatis-sqlsessionfactory7.jpg)

###### `mapper` 接口

![](/img/in-post/mybatis-sqlsessionfactory8.jpg)

###### `mapper` 配置文件

![](/img/in-post/mybatis-sqlsessionfactory9.jpg)

###### 测试类

![](/img/in-post/mybatis-sqlsessionfactory10.jpg)




**这张图是由 `XMLStatementBuilder` 解析出来的一个select节点的信息（`mapperedStatement`）

![](/img/in-post/mybatis-sqlsessionfactory.jpg)

> 每一个增删改查的操作就是一个mapperedStatement


**最终解析完成的configuration**

![](/img/in-post/mybatis-sqlsessionfactory1.jpg)

>  `configuration` 中保存了所有配置文件中详细信息

 **`configutation` 中的 `mapperRegistry` 保存了 `konwnMappers`**
 
> `konwnMappers` 保存了 `mapper` 接口和对应的 `mapper` 代理工厂的对应
 
 ![](/img/in-post/mybatis-sqlsessionfactory2.jpg)
 
 
 **最终返回 `sqlSessionFactory` 接口的实现类 `DefaultSqlSessionFactory`**
 
 ![](/img/in-post/mybatis-sqlsessionfactory3.jpg)
 
 ![](/img/in-post/mybatis-sqlsessionfactory4.jpg)
 
 > 最终将Configuration 返回给 `SqlSessionFactoryBuilder` 的 `build` 方法 ，build创建了 `DefaultSqlSessionFactory` 实例返回给 `sqlSessionFactory`
 
 