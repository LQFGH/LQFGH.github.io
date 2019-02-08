---
layout:     post
title:      "MyBatis-原理"
subtitle:   "查询实现"
date:       2019-02-07 11:10
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
  
  ![](/img/in-post/mybatis-select1.jpg)

* `Configuration`:保存`configuration`配置文件中所有的配置文件
* `DefaultSqlSessionFactory` 创建`SqlSession` 的工厂类
* `Executor` 是执行增删改查等方法最基础的方法，底层执行增删改查还是要调用`Executor`的类的方法的实现
* `MapperMethod` 存放`mappe` 中rselect|delete|update|insert标签的解析信息


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

![](/img/in-post/mybatis-select.jpg)

> 测试类中的 `openSession()`处打断点，作为程序的入口
> 

***


## 断点走走

###### 执行代理对象的 `invoke` 方法

![](/img/in-post/mybatis-select2.jpg)

> 45 行，判断若调用的方法是继承自 `Object` 的方法，则直接执行，并将结果返回
> 
> 若执行接口自定义的方法，则不直接执行 `invoke` 方法
> 
> 52行从缓冲中获取要执行的方法的 `mapper` 配置文件解析的信息
> 
> 接下来进入52行获取 `MapperMethod` ，该方法就在当前方法下面
> 

###### 获取 `MapperMethod` 

![](/img/in-post/mybatis-select3.jpg)

> 57行从缓存中获取 获取 `select` 标签解析信息
> 
> 若缓存中没有 对应的 `select` 标签解析的 `MapperMethod` 则创建 `MapperMethod` 
> 
> 接下来进入59 行看看创建 `MapperMethod` 都做了哪些事情
> 

**1、创建 `MapperMethod`** 

![](/img/in-post/mybatis-select5.jpg)

> 48行创建 `SqlCommand` 对象，`SqlCommand` 是`MapperMethod`的静态内部类
> 49行创建 `MethodSignature`对象，`MethodSignature` 是`MapperMethod`的静态内部类

**1.1 创建 `SqlCommand`** 

![](/img/in-post/mybatis-select6.jpg)

> 进入48行
> 
> 208 行创建 全类名+方法名，在后面用于在 `configuration` 中获取 `MapperMethod`
> 
> statementName 值为 `com.mybatis.source.dao.EmployeeMapper.selectEmployeeById`
> 
> 211 行从 `configuration` 中的 `mappedStatements`（map） 中获取`MappedStatement`
> 
> 226 行将 `statementName` 值设置给 `SqlCommand`  类中的属性 `name`
> 
> 将 `getSqlCommandType` 赋值给 `SqlCommand`  类的 `type`
> 
> 至此，`SqlCommand`创建完成
> 

**1.2 创建 `MethodSignature`**

![](/img/in-post/mybatis-select7.jpg)

> 至此创建 `MapperMethod` 完成
> 

> 接下来返获取 `MapperMethod` 中
> 
> 60行将获取到的 `MapperMethod` 放入缓存中
> 接下来回到 `invoke` 方法 


###### 调用 `MapperMethod` 的 `execute`方法

![](/img/in-post/mybatis-select8.jpg)

> `execute` 方法判断执行的 `sql` 类型是什么
> 
> 当前执行的查询，所以进入70行判断
> 
> 判断返回的类型是多条还是Map还是游标，由于返回的是单条记录，索引进入else判断
>
>  81 行用于分装参数，将参数封装到 `map` 中,接下来进入到改方法中
> 

###### 参数封装 `convertArgsToSqlCommandParam`

代码这里就不贴出来了，读者可以看看类 `ParamNameResolver` 的代码将参数封装为 `map`的逻辑

###### 调用`sqlSession.selectOne` 方法

![](/img/in-post/mybatis-select9.jpg)

> 77行实际上调用的是查询多个记录，但是最终返回的是一条记录
>
> 进入77行
> 


