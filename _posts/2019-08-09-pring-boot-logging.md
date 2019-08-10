---
layout:     post
title:      "springboot - 日志使用"
subtitle:   "spring boot日志配置及使用"
date:       2019-08-09 11:10
author:     "LQFGH"
header-img: "img/spring-cloud-bg.png"
header-mask: 0.3
catalog:    true
tags:
  - springboot
---

***

> 今天我们就来说说 `spring boot` 的日志使用
> 

#### 1、日志的输出格式

%d表示日期时间，
%thread表示线程名，
%-5level：级别从左显示5个字符宽度
%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
%msg：日志消息，
%n是换行符

**举例**

```xml
%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
<!-- 输出：
2019-08-09 23:35:39.591 [qtp1803452728-47] INFO  c.f.b.c.feng.provider.controller.eurekaController - Logging message
```


#### 2、 配置日志信息

```properties
# 配置 `com.feng` 包的日志输出级别
logging.level.com.feng=debug
# 不指定路径在当前项目下生成springboot.log日志，/spring/logs表示磁盘更路径下
logging.path=/spring/logs
# 可以指定完整的路径；
logging.file=C:/springboot.log
# 控制台日志输出格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
```

| logging.file | logging.path | Example  | Description             |
| ------------ | ------------ | -------- | ----------------------- |
| (none)       | (none)       |          | 只在控制台输出                 |
| 指定文件名        | (none)       | my.log   | 输出日志到my.log文件           |
| (none)       | 指定目录         | /var/log | 输出到指定目录的 spring.log 文件中 |

#### 3、指定配置

> 给类路径下放上每个日志框架自己的配置文件即可；<u>SpringBoot</u>就不使用他默认配置的了,
> 
| Logging System          | Customization                            |
| ----------------------- | ---------------------------------------- |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`      |
| `JDK (Java Util Logging)` | `logging.properties`     


> `logback.xml`：不带配置文件 `-spring` 直接就被日志框架识别了，不能使用 `spring` 的 `Profile` 功能
>
```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>

```

如：

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>
```


#### 4、切换日志框架

> `spring boot`  默认使用 `slf4j+logback` 的组合,所以配置 `spring boot` 日志直接加入 `logback` 配置文件即可


![](../img/in-post/1565366105(1).jpg)


> 接下来我们将logback切换为 `slf4j+log4j`，这里只是为了切换而切换，实际上将日志切换为 `log4j` 毫无意义
> 

**slf4j+log4j**
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>

```


> 切换为 `log4j2` 只需要将 `logback` 和 `log4j-over-slf4j` 依赖排除，并导入 `slf4j-log4j12` `log4j2` 对 `slf4j` 
> 的实现
> 
> 接下来直接使用 `log4j`
> 

**log4j2**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

> 排除 `spring-boot-starter-logging` 导入 `spring-boot-starter-log4j2`
-----------------

