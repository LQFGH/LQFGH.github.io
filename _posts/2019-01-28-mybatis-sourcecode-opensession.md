---
layout:     post
title:      "MyBatis-原理"
subtitle:   "SqlSession创建过程"
date:       2019-01-27 11:10
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
> 其实创建 `SqlSession` 的过程主要是创建 `Executor` 实例
> 
> **在这个过程中还是希望读者可以跟随本文打断点亲自看下源码，体会很更深些**
> 

  **`SqlSession` 初始化过程时序图**
  
  ![](/img/in-post/mybatis-sqlsession.jpg)

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

![](/img/in-post/mybatis-sqlsession1.jpg)

> 测试类中的 `openSession()`处打断点，作为程序的入口
> 

***


## 断点走走



###### 调用 `openSession()` 方法

![](/img/in-post/mybatis-sqlsession2.jpg)

> **注意！** 这里的的代码在 `DefaultSqlSessionFactory` 中
> 
> 47行处调用当前类中的 `openSessionFromDataSource` 方法,
> 接下来进入 `openSessionFromDataSource` 方法


###### 一个很重要的方法  `openSessionFromDataSource`

![](/img/in-post/mybatis-sqlsession3.jpg)

> 为什么说这是一个很重要的方法呢？因为这个方法概括了创建  `SqlSession` 
> 都做了哪些工作。
> 
> 94、95行创建事务管理器
> 
> 96行是最核心的一个步骤，创建 `Executor` 对象
> 
> 97行创建 `DefaultSqlSession` 对象，也就是 `SqlSession` 接口的实现类
>
> 那接下来就进入96行看看到底是怎么创建的 `Executor`
> 

###### 创建对象 `Executor`


**首先我们看下 `Executor` 是干什么用的**

![](/img/in-post/mybatis-sqlsession5.jpg)

> 由 `Executor` 的方法可以看出，它是执行增删改查等方法最基础的方法，底层执行增删改查还是要调用
>  `Executor`的类的方法的实现


![](/img/in-post/mybatis-sqlsession4.jpg)

> 进入 `newExecutor` 方法
> 
> 首先看下参数 `ExecutorType` 是从一开始就跟着我们到了这里，到底是干什么的

![](/img/in-post/mybatis-sqlsession6.jpg)

> `ExecutorType` 是一个枚举类，分别表示 批量模式，简单模式，可重用处理模式，
> 不同模式适合不同的场景
> 
> 我们这里是 `simple` 简单模式
> 
> 知道了`ExecutorType` 是干什么的，我们接着看 `newExecutor` 方法
> 
> 541 - 549 行主要是判断创建那种模式的 `ExecutorType`
> 
> 551行，如果有使用二级换从则创建使用二级缓存的 `Executor`，其实只是用
>  `CachingExecutor`包装了的 `executor` 对象，只是在执行查询的时候会先去缓存中查询
>  
> 553 行，执行所有插件的拦截器对 `executor` 进行处理
> 
> 到这里 `executor` 对象创建完成了(最重要的东西创建完成了)
> 
> 那我们再回到 `DefaultSqlSessionFactory` 中
> 

###### 返回 `DefaultSqlSessionFactory` 

> 返回最初 `DefaultSqlSessionFactory` 中的 `openSessionFromDataSource`方法
> 
> 97行，将 整个mybatis生命周期都很重要的 `configuration` 和 `创建SqlSession` 很重要的 
> `executor` 对象 作为创建 `DefaultSqlSession` 的参数构造 `DefaultSqlSession`
> 并作为 `SqlSession` 的实例返回