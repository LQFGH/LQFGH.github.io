---
layout:     post
title:      "MyBatis-原理"
subtitle:   "sqlSessionFactory初始化原理"
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


>  1、整个 `sqlSessionFactory` 的过程就是把 配置文件的信息解析并存放到 `Configuration` 中，并创建`DefaultSqlSessionFactory` 实例返回给 `sqlSessionFactory`
> 
>  2、`configuration` 贯穿整个初始化的流程，保存了所有配置文件的详细信息
>  


  **`sqlSessionFactory` 初始化过程时序图**
  
  ![](/img/in-post/mybatis-sqlsessionfactory5.jpg)

  * `SqlSessionFactoryBuilder`:创建`SqlSessionFactory`
* `Configuration`:保存`configuration`配置文件中所有的配置文件
* `XMLConfigBuilder`：解析`Configuration`配置文件
* `XMLMapperBuilder`：解析`Mapper`配置文件
* `XMLStatementBuilder`：解析增删改查标签
* `XPathParser`：具体解析节点信息的类
* `MapperBuilderAssistant`:是`XMLMapperBuilder`解析`Mapper.xml`时用到一个帮助类,实现一部分逻辑处理和处理过程中的上下文保存。

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

![](/img/in-post/mybatis-sqlsessionfactory10.jpg)

> 测试类中的创建sqlSessionFactory处打断点，作为程序的入口
> 

***

## 断点走走


> 代码的整个流程都涉及到了，读者可以自己跟随本文进行断点阅读代码
> 
> 需要注意的代码会用会标注出来
> 
> 另外特别注意下截图的代码是在哪一个类中
> 
> 请结合第一部分 `总结部分` 看接下来的内容，更容易理解

###### 调用 `SqlSessionFactoryBuilder` 的build方法创建 `SqlSessionFactory`

![](/img/in-post/mybatis-sqlsessionfactory11.jpg)

> 注意下 `SqlSessionFactoryBuilder` 的方法就可以看出该类只是用于创建`SqlSessionFactory`，
> 
> 标出的第77行为创建**全局配置**文件解析的类，这里不多做介绍。重点是调用第78行中的
> 
>  `parser.parse()`调用，进行全局配置文件解析。

###### 调用 `XMLConfigBuilder` 的全局配置文件解析方法
![](/img/in-post/mybatis-sqlsessionfactory12.jpg)

```java
 if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
  }
 parsed = true;
```

> 代码用于判断当前 `XMLConfigBuilder` 实例是否执行过，每个实例只能执行一次，因为全局配置文件指只会解析一
> 次即可，如果没有解析过就将标志置为`true`,开始解析。

```java
  parseConfiguration(parser.evalNode("/configuration"));
```

> 这一行代码调用当前类中解析配置文件的方法 `parser.evalNode("/configuration")` 是
> 
> 解析全局配置文件中 `configuration` 节点，parseConfiguration方法就在当前方法下面。


###### 解析全局配置文件 

![](/img/in-post/mybatis-sqlsessionfactory13.jpg)

> 该方法主要用来解析全局配置文件中的每一个标签，其他标签不做说明，`mapper` 标签的解析说明下。
> 
> 接下来进入118行的方法

###### 解析 `mapper` 标签

![](/img/in-post/mybatis-sqlsessionfactory14.jpg)

> 357行代码由于我的全局配置文件中 `mapper`配置的是 `resource`所以直接进入 `else`
> 
> 366行根据 `mapper`标签的 `resource`获取的 `mapper`配置文件的路径，将 `mapper`一流的形式读取
> 
> 367行创建 `mapper`配置文件解析器
> 
> 接下来进入368行接下 `mapper`配置文件
> 

###### 进入`mapper`配置文件解析

![](/img/in-post/mybatis-sqlsessionfactory15.jpg)

> **这里注意下**，代码已经从 `XMLConfigBuilder` 进入到了  `XMLMapperBuilder`
> 
> 进入到92行方法看看具体解析mapper配置文件

######  `mapper`配置文件的解析

![](/img/in-post/mybatis-sqlsessionfactory16.jpg)

> 可以看到这个方法是对 `mapper`配置文件的各个标签解析，我们进入118行看看具体的增删改查的解析。
> 


###### 调用 `select|insert|update|delete` 解析

![](/img/in-post/mybatis-sqlsessionfactory17.jpg)

> 125行处由于我没有配置 `databaseId` 所以获取到 `configuration.getDatabaseId()`
> 
> 为空，不进入判断。
> 
> 那么接下来进入128行进行 `select|insert|update|delete` 解析到图中131行方法
> 
> 133行创建 `select|insert|update|delete` 解析器
> 
> 135行进入正真的 `select|insert|update|delete`解析


###### 解析 `select|insert|update|delete` 

**这个方法有比较长，只截了下半部分，上半部分是对标签中属性的解析**

![](/img/in-post/mybatis-sqlsessionfactory18.jpg)

> **这里注意下**，从这里已经进入到 `builderAssistant` 类中
> 
> 109行将所有解析的属性添加到 `builderAssistant` 中(最终都会保存到 `configuration`中)
> 
> 进入到109中保存属性方法中
> 

###### 属性去了哪里

![](/img/in-post/mybatis-sqlsessionfactory19.jpg)

> 图中302行发现最终还是将所有属性解析出来的属性都保存到了 `configuration`中
> 
> 至此解析完成，接下来创建 `SqlSessionFactory`

###### 返回到`SqlSessionFactoryBuilder` 类中

![](/img/in-post/mybatis-sqlsessionfactory20.jpg)

> 我们回到最初被调用的 `SqlSessionFactoryBuilder` 中的
> 
> 78行调用的 `build`方法其实调用的是下面91行重载的 `build`方法
> 
> 92行返回一个 `DefaultSqlSessionFactory`实例，并将辛辛苦苦解析出来的 `configuration`
> 作为参数
> 
> 其实 `DefaultSqlSessionFactory`是 `SqlSessionFactory`的实现类，`SqlSessionFactory`
> 是一个接口，这里将 `DefaultSqlSessionFactory`作为实例返回给`SqlSessionFactory`



***


## 附加截图


###### 这张图是由  `XMLStatementBuilder` 解析出来的一个select节点的信息（ `mapperedStatement`）

![](/img/in-post/mybatis-sqlsessionfactory.jpg)

> 每一个增删改查的操作就是一个 `mapperedStatement`


###### 最终解析完成的 `configuration`

![](/img/in-post/mybatis-sqlsessionfactory1.jpg)

>  `configuration` 中保存了所有配置文件中详细信息
>  

######  `configutation` 中的  `mapperRegistry` 保存了  `konwnMappers`
  
 ![](/img/in-post/mybatis-sqlsessionfactory2.jpg)
 
> `konwnMappers` 保存了 `mapper` 接口和对应的 `mapper` 代理工厂的对应
 
 