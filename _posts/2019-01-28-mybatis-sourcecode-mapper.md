---
layout:     post
title:      "MyBatis-原理"
subtitle:   "创建mapper代理对象"
date:       2019-02-01 11:10
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MyBatis
  - Source Code
---

***


## 总结


> **这里将总结放到前面的目的就是希望读者可以先了解整个的流程和重要代码的意义，以免在具体看的时候迷失方向**
> 
> **在这个过程中还是希望读者可以跟随本文打断点亲自看下源码，体会很更深些**
> 

**`SqlSession` 初始化过程时序图**
  
  ![](/img/in-post/mybatis-mapper10.jpg)

* `Configuration`:保存`configuration`配置文件中所有的配置文件
* `DefaultSqlSessionFactory` 创建`SqlSession` 的工厂类
* `Executor` 是执行增删改查等方法最基础的方法，底层执行增删改查还是要调用`Executor`的类的方法的实现

***


## 简单的运行代码


> 由于是看源码，这里的只有两个查询方法和非常简单的配置


###### 全局配置文件

![](/img/in-post/mybatis-sqlsessionfactory7.jpg)

###### `mapper` 接口

![](/img/in-post/mybatis-sqlsessionfactory8.jpg)

###### `mapper` 配置文件

![](/img/in-post/mybatis-sqlsessionfactory9.jpg)

###### 测试类

![](/img/in-post/mybatis-mapper.jpg)

> 测试类中的 `sqlSession.getMapper`处打断点，作为程序的入口
> 

***

## 断点走走


###### 调用  `sqlSession.getMapper` 

![](/img/in-post/mybatis-mapper1.jpg)

> 进入` sqlSession.getMapper(EmployeeMapper.class)` 方法看到调用的是 `sqlSession`
> 
> 中 `configuration` 的 `getMapper` 方法
> 

###### 调用  `configuration` 的 `getMapper` 方法


![](/img/in-post/mybatis-mapper2.jpg)

> 我们发现在 `configuration` 中调用的是其属性 `mapperRegistry` 的 `getMapper`
> 方法，那 `mapperRegistry`是什么呢?
> 

![](/img/in-post/mybatis-mapper3.jpg)

> 实际上 `mapperRegistry`中存放的是接口和创建其代理的工厂的 `knownMappers`
> 
>   `mapperRegistry` 是创建 `sqlSesssionFactory` 时创建的
>  
> 接下来进入 `getMapper` 方法
> 

###### `MapperProxyFactory`  中获取 `MapperProxy` 代理

![](/img/in-post/mybatis-mapper4.jpg)

> 正如上面所说，45 行从 `knownMappers` 中的 `knownMappers` 中获取 `MapperProxyFactory`
>
>  50 行 从 `MapperProxyFactory` 中获取 `MapperProxy` 对象
>  
> 接下来进入 `MapperProxyFactory` 
> 


###### `MapperProxyFactory`  中创建 `MapperProxy` 代理

![](/img/in-post/mybatis-mapper5.jpg)


> 图中51 行处创建  `MapperProxy` 对象
> 
> 52 行调用46行的 `newInstance` 方法
> 
> 47行调用原生的`jdk` 的 `Proxy` 类创建代理对象
> 
> 接下来进入51 行看看 `MapperProx` 是什么
> 


###### `MapperProxy` 代理类


![](/img/in-post/mybatis-mapper6.jpg)

> 图中30 行，`MapperProxy` 实现了 jdk 原生的代理接口
> 
> 实现了 `invoke` 方法，在调用代理对象的方法时会调用 `invoke`方法
> 
> 至此创建代理对象完毕，一步步将代理对象返回
> 

