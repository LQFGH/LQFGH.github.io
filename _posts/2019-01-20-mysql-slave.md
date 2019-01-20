---
layout:     post
title:      "MySql高级技术-mysql主从复制"
subtitle:   "MySQL主从复制配置"
date:       2019-01-20 01:55:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql高级
---

## `MySQL` 主从复制步骤

***


*  一、`master` 将修改记录到二进制日志 `binary log` 中. 这些记录过程叫做二进制日志事件`binary log events`
*  二、`slave` 将 `master` 的中继日志 `binary log events` 拷贝到它的中继之日中 `relay log`
*  三、`slave` 重做中继日志的事件，将改变写到自己的数据库中。`MySQL` 的主从复制是异步且串行化的  


## 主从复制的基本原则

***

* 每个 `slave` 只有一个 `master` 
* 每个 `slave` 只有一个唯一的服务器 `ID`
* 每个 `master` 可以有多个 `slave


## 主从复制最大的问题-延时

***


## 配置主从复制

***

> 我采用的主机是 `windows`  ，两台从机是安装在虚拟机上 `linux`
> `master` 的 `IP`: `192.168.1.102`   端口号：`3306`
> 两台从机的端口都是 `3306`,`IP` 地址分别是：`192.168.174.128`、`192.168.174.129`   



