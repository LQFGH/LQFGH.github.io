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
> **查询流程是mybatis最核心的东西**
> 
> **在这个过程中还是希望读者可以跟随本文打断点亲自看下源码，体会很更深些**
> 

######   `SqlSession` 初始化过程时序图
  
  ![](/img/in-post/mybatis-select1.jpg)
  
######   查询流程
  
  ![](/img/in-post/mybatis31.jpg)

###### 具体类的含义

* `Configuration`:保存`configuration`配置文件中所有的配置文件
* `DefaultSqlSessionFactory` 创建`SqlSession` 的工厂类
* `Executor` 是执行增删改查等方法最基础的方法，底层执行增删改查还是要调用`Executor`的类的方法的实现
* `MapperMethod` 存放`mappe` 中rselect|delete|update|insert标签的解析信息
* `StatementHandler`：处理 `sql` 语句预编译，设置参数等相关工作；
* `ParameterHandler`：设置预编译参数用的
* `ResultHandler`：处理结果集
* `TypeHandler`：在整个过程中，进行数据库类型和 `javaBean`类型的映射


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


###### 调用`sqlSession.selectOne` 方法（重点的查询从这里开始）

![](/img/in-post/mybatis-select9.jpg)

> 77行实际上调用的是查询多个记录，但是最终返回的是一条记录
>
> 进入77行
> 


###### 调用 `selectList` 方法

![](/img/in-post/mybatis-select11.jpg)

> 147行从 `configuration` 中获取 `MappedStatement`
> 
> 148行执行 `executor` 的查询方法
> 
> 接下来进入148行 `executor` 的查询方法
>
> 图中 `wrapCollection(parameter)` 解释请看 `延申部分 一`
> 

###### 执行 `executor` 的 `query` 方法

![](/img/in-post/mybatis-select13.jpg)

> 81行获取 `sql` 的详细信息，详细请看 `延申部分 二`
>
> 82行创建缓冲的key
>
> 83行执行查询,进入83行
> 

![](/img/in-post/mybatis-select15.jpg)

> 96-108行当有缓存是执行，我们现在没有用到缓存，所以直接执行109行
> BaseExecutor的查询方法，接下来进入109行
> 

![](/img/in-post/mybatis-select16.jpg)

> 152 行从一级缓存中获取查询结果，但是我们是第一次执行，所以直接执行156行，
> 从数据库中查询，接下来进入 `queryFromDatabase` 方法
>

![](/img/in-post/mybatis-select17.jpg)

> 322,326,328行将查询的结果放入到一级缓存中
> 
> 324行执行 `doQuery` 方法
> 

###### 执行 `SimpleExecutor` 的 `doQuery` 方法

![](/img/in-post/mybatis-select18.jpg)

> 这个地方出现了mybatis的四大对象之一 `StatementHandler`,其实在创建`StatementHandler`
> 时将另外两大对象（ `ParameterHandler` 和 `ResultSetHandler` ）也
> 作为 `StatementHandler`的成员一起创建了，都是调用了 `Configuration`
> 中的对应的创建方法创建的，另外说下 `executor` 也是在 `Configuration` 创建的。
> 
> 创建 `StatementHandler` 的过程请看 `延伸部分 三`
> 
> 62行预编译参数，其实调用的是 `ParameterHandler` 进行参数预编译，详细请看 `延伸部分 四`。
> 
> 63行调用处理结果集，其实调用的是 `ResultSetHandler` 对结果集进行处理。
> 
> 还需要注意的地方就是每当创建四大对象时就会调用 `interceptorChain.pluginAll` 
> 对其进行拦截处理，这也正是 `mybatis` 插件的原理
>


***


## 延申部分


###### 一： `wrapCollection(parameter)`

![](/img/in-post/mybatis-select12.jpg)

> 如果参数的类型是 `Collection` `list` `array` 则按照对应的名字封装
> 

###### 二：`boundSql` 详细信息截图

![](/img/in-post/mybatis-select14.jpg)

> sql语句及其参数的解析就是在这里完成的
> 

###### 三：`StatementHandler`创建过程

![](/img/in-post/mybatis-select20.jpg)

> 当前方法是 `configuration` 中的 `newStatementHandler` 方法
>
> 530行创建 `StatementHandler` 的实现类 `RoutingStatementHandler`
>
> 531 行调用插件
> 
> 接下来进入530行
> 

![](/img/in-post/mybatis-select21.jpg)

> 41行判断查询的类型，创建 `StatementHandler`，由于我们使用的是默认的 `StatementType` 为
> `PREPARED` ，所以直接执行46行
>
> 接下来进入46行创建 `PreparedStatementHandler`
> 

![](/img/in-post/mybatis-select22.jpg)


> 40行调用其继承自 `BaseStatementHandler` 的构造方法，接下来进入
> `BaseStatementHandler` 的构造方法。
> 

![](/img/in-post/mybatis-select24.jpg)

> 69行70行调用configuration中的方法创建 `ParameterHandler` 和 `ResultSetHandler`
> 作为 `StatementHandler` 的参数
>


###### 四： `ParameterHandler` 进行参数预编译

> 进入 62行
> 

![](/img/in-post/mybatis-select25.jpg)

> 86 调用 `StatementHandler` 的参数处理方法，接下来进入86行
>

![](/img/in-post/mybatis-select26.jpg)

> 直接进入
> 

![](/img/in-post/mybatis-select27.jpg)

> 在这里就调用了 `parameterHandler` 的参数处理方法
> 继续进入
> 

![](/img/in-post/mybatis-select28.jpg)

> 87调用 `typeHandler` 对参数进行预编译
>

![](/img/in-post/mybatis-select29.jpg)

> 44行获取当前参数对应的 `typeHandler` 对参数进行预编译，
> 进入45行看看是怎么预编译的
> 

![](/img/in-post/mybatis-select30.jpg)
  
> 31行，实际上调用的jdbc原生的PreparedStatement 对参数进行预编译
>
