---
layout:     post
title:      "spring-boot-启动配置原理"
subtitle:   "启动原理、运行流程、自动配置原理"
date:       2019-08-24 17:10
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:

  - springboot
---





[TOC]



# Ⅰ创建 `SpringApplication` 对象



这是一个 `spring boot` 启动类 `run` 方法

```java
public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
		return new SpringApplication(sources).run(args);
}
```

> 首先创建了一个 `SpringApplication` 类，我们看下是怎么创建的



```java
	public SpringApplication(Object... sources) {
		initialize(sources);
	}
```

> 接下来进入 `initialize` 方法



```java
	private void initialize(Object[] sources) {
		if (sources != null && sources.length > 0) {
			this.sources.addAll(Arrays.asList(sources));
		}
		this.webEnvironment = deduceWebEnvironment();
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
		setListeners((Collection)
                     getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

> 该方法中分别从所有 `spring` 的 `jar` 包的 `META-INF/spring.factories` 文件中分别获取了 所有类型为`ApplicationContextInitializer` 和 `ApplicationListener` 的类添加到当前类中。然后将在方法调用栈中获取了 带有 `main` 方法的主配置类，也就是我们自己定义的 `ConfigurableApplicationContext` 类



