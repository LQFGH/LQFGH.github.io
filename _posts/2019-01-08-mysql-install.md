---
layout:     post
title:      "MySql高级技术-mysql变量"
subtitle:   "mysql安装"
date:       2019-01-08 19:00:00
author:     "LQFGH"
header-img: "img/post-mysql.jpg"
header-mask: 0.3
catalog:    true
tags:
  - mysql高级
---

> linux 下查看安装目录 ps 0ef | grep mysql

#### **mysql目录**

| 路径                | 解释               | 备注                      |
| ----------------- | ---------------- | ----------------------- |
| /var/lib/mysql/   | mysql数据库的存放路径 |                         |
| /usr/share/mysql  | 配置文件目录           | mysql.server命令及配置文件     |
| /usr/bin          | 相关命令目录           | mysqladmin mysqldump等命令 |
| /etc/init.d/mysql | 相关起停脚本           |                         |


#### **配置文件**
windows下配置文件是my.ini
linux下配置文件是my.cof

| 配置    |  作用   |  注释  |
| --- | --- | ---|
|  log-bin=d:/XXX   |  主从复制   | |
|  log-err=d:/xxx   |  错误日志   | 默认关闭 |

#### 数据文件

###### 数据库文件
windows:d:\devinstall\mysqlxxx\data
linux /var/lib/mysql

**位于data中个表的文件夹中**
frm：存放表结构
myd：从放数据
myi：存放表索引

