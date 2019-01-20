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


> 主从复制的机器最好都能保证各服务器的 `MySQL` 的版本都一样，我这里都用 `5.7.24`
> 
> 如果启动 `MySQL` 有问题可以用 `mysqld --console` 命令排查错误

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

> 配置之前请确保主机和从机之间可以互相联通，并且开放端口
> 我采用的主机是 `windows`  ，两台从机是安装在虚拟机上 `linux`
> `master` 的 `IP`: `192.168.1.102`   端口号：`3306`
> 两台从机的端口都是 `3306`,`IP` 地址分别是：`192.168.174.128`、`192.168.174.129`   


###### 主机 `windows` 修改 `my.ini` 配置文件

> 一下所有的配置都是在 `mysqld` 下

**一、【必须】 主服务器 `ID` 唯一**

``` my.ini
server-id=1
```

> 这里的 `server-id` 是一定要配置的，且是唯一的，我这里配置为 `1`


**二、【必须】 启用二进制日志**

```my.ini
log-bin=D:\DevInstall\mysql-5.7.24-winx64\data\mysqlgin
```


**三、【可选】 启用错误日志**

```my.ini
log-err=D:\DevInstall\mysql-5.7.24-winx64\data\mysqlerr
```

> 由于我启动的时候总是报这个变量错误，所以这个就不要配置了

**四、【可选】根目录**

```my.ini
basedir = D:\\DevInstall\\mysql-5.7.24-winx64
```


**五、【可选】临时目录**

```my.ini
D:\\DevInstall\\mysql-5.7.24-winx64
```


**六、【可选】数据目录**

```my.ini
datadir = D:\\DevInstall\\mysql-5.7.24-winx64\\data
```


**七、【可选】配置 `master` 即可读也可以写**

```my.ini
read-only=0
```


**八、【可选】设置不需要复制的数据库**

```my.ini
binlog-ignore-db=mysql
```


**九、【可选】配置需要复制的数据库**

```my.ini
binlog=do-db=xxx
```

> 八、九二选一



###### 从机 `Linux` 修改 `my.cnf` 配置文件


**一、【必须】 从服务器 `ID` 唯一**

```my.cnf
server_id = 2
```


**二、【启用二进制日志文件】**

```my.cnf
log-bin=mysql-bin
```


> **！重要**
> 
> 配置完成所有的配置之后一定要重新启动 `windows` 和 `linux` 上的 `MySQL` 服务



## `windows` 主机上建立账户并授权 `slave`

***

###### **授权**

```sql
GRANT REPLICATION SLAVE ON *.* TO 'root'@'192.168.174.128' IDENTIFIED BY 'xxx';
```

###### **刷新配置**

```sql
flush privileges;
```


###### **查询 `master` 的状态**

```sql
show master status;
```

![](/img/in-post/mysql-slave.jpg)

- [ FIle] 二进制日志文件
- [ Position] 开始记录的位置
- [ Binlob_Do_Db] 要复制到数据库，没有配置代表新建的数据库都会被复制
- [ Binlog_Ignore_Db] 不需要复制的数据库

> 记下这里的 `File` 和 `Position` ,后面需要用到

**`windows`的配置到这里就结束了**



## `Linux` 从机上配置需要复制的主机

***


###### 配置 `master` 信息

```sql
CHANGE MASTER TO MASTER_HOST='192.168.1.102',
MASTER_USER='root',
MASTER_PASSWORD='xxx',
MASTER_LOG_FILE='mysqlbin.000001',MASTER_LOG_POS=595;
```

> 这里的 `MASTER_LOG_FILE` 和 `MASTER_LOG_POS` 就是上面 `master` 运行 `show master status;` 查询出来的结果

###### 启动从机复制功能

```sql
start slave;
```

 > 停止主从复制 `stop slave;`

###### 查看是否配置成功

```sql
show slave status\G;
```

![](/img/in-post/mysql-slave1.jpg)

> 图中红色部分必须显示 `yes` 才行



## 测试主从复制

**测试可以在master 上创建个数据库创建个表插个数据，然后再到slave上看看有没有复制。**

