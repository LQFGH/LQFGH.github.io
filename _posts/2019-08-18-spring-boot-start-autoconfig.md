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



# Ⅰ启动流程

重要的四个组件

#### 一：准备环境

- 执行 `ApplicationContextInitializer.initialize` 方法
- 监听器 `SpringApplicationRunListener` 回调 `contextPrepared`方法加载主配置类信息
- 监听器 `SpringApplicationRunListener` 回调  `contextLoaded` 方法

> `ApplicationContextInitializer` 和  `SpringApplicationRunListener`  需要在 `META-INF/spring.factories` 中定义才能生效

#### 二：刷新启动 `IOC` 容器 

- 扫描加载容器中所有组件
- 包括从 `META-INF/spring.factories` 中获取的所有的 `EnableAutoConfiguration` 组件

#### 三：回调容器中所有 `ApplicationRunner` 、`CommandLineRunner` 组件

#### 四：监听器 `SpringApplicationRunListener` 回调 `finished`

------



# Ⅱ创建 `SpringApplication` 对象



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
            //保存主配置类
			this.sources.addAll(Arrays.asList(sources));
		}
        //判断当前是否一个web应用
		this.webEnvironment = deduceWebEnvironment();
          //从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
		//从类路径下找到ETA-INF/spring.factories配置的所有ApplicationListener
        setListeners((Collection)
                     getSpringFactoriesInstances(ApplicationListener.class));
         //从多个配置类中找到有main方法的主配置类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

> 该方法中分别从所有 `spring` 的 `jar` 包的 `META-INF/spring.factories` 文件中分别获取了 所有类型为`ApplicationContextInitializer` 和 `ApplicationListener` 的类添加到当前类中。然后将在方法调用栈中获取了 带有 `main` 方法的主配置类，也就是我们自己定义的 `ConfigurableApplicationContext` 类

------



# Ⅲ `run` 方法启动过程

这段代码就是 `spring boot` 启动的整个流程

```java
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		FailureAnalyzers analyzers = null;
		configureHeadlessProperty();
        //获取SpringApplicationRunListeners；从类路径下META-INF/spring.factories
		SpringApplicationRunListeners listeners = getRunListeners(args);
        //回调所有的获取SpringApplicationRunListener.starting()方法
		listeners.starting();
		try {
             //封装命令行参数
			ApplicationArguments applicationArguments = new
                DefaultApplicationArguments(args);
             //准备环境
            //创建环境完成后回调SpringApplicationRunListener.environmentPrepared()；表示环境准备完成
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			Banner printedBanner = printBanner(environment);
            //创建ApplicationContext；决定创建web的ioc还是普通的ioc
			context = createApplicationContext();
			analyzers = new FailureAnalyzers(context);
            //准备上下文环境;将environment保存到ioc中；而且applyInitializers()；
       		//applyInitializers()：回调之前保存的所有的ApplicationContextInitializer的				initialize方法
      		//回调所有的SpringApplicationRunListener的contextPrepared()；
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
            //prepareContext运行完成以后回调所有的SpringApplicationRunListener的					contextLoaded（）；
            
            //刷新容器；ioc容器初始化（如果是web应用还会创建嵌入式的Tomcat）；Spring注解版
       		//扫描，创建，加载所有组件的地方；（配置类，组件，自动配置）
			refreshContext(context);
            //从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
       		//ApplicationRunner先回调，CommandLineRunner再回调
			afterRefresh(context, applicationArguments);
             //所有的SpringApplicationRunListener回调finished方法
			listeners.finished(context, null);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
            //整个SpringBoot应用启动完成以后返回启动的ioc容器；
			return context;
		}
		catch (Throwable ex) {
			handleRunFailure(context, listeners, analyzers, ex);
			throw new IllegalStateException(ex);
		}
	}
```

------



# Ⅳ时间监听机制



```java
public class HelloApplicationContextInitializer implements
    ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("HelloApplicationContextInitializer...initialize...");
    }
}
```



```java
public class HelloSpringApplicationRunLister implements SpringApplicationRunListener {   
    public HelloSpringApplicationRunLister(SpringApplication application, String[]
                                           args) {}
    @Override
    public void starting() {
        System.out.println("HelloSpringApplicationRunLister...starting...");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        System.out.println("HelloSpringApplicationRunLister...environmentPrepared...");
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("HelloSpringApplicationRunLister...contextPrepared...");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("HelloSpringApplicationRunLister...contextLoaded...");
    }

    @Override
    public void finished(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("HelloSpringApplicationRunLister...finished...");
    }
}
```

> 这里需要注意的是 `HelloSpringApplicationRunLister` 必须要实现构造方法，参数是固定的 `SpringApplication` 和  `String[]`



`ApplicationContextInitializer` 和 `SpringApplicationRunListener` 组件需要被配置在 `META-INF/spring.factories` 才能生效，`spring ` 中的很多组件都是这样配置的

```properties
org.springframework.context.ApplicationContextInitializer=\
com.feng.boot.datajpa.listener.HelloApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=\
com.feng.boot.datajpa.listener.HelloSpringApplicationRunLister
```



`ApplicationRunner` 和 `CommandLineRunner` 组件只需要添加到容器中即可生效

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("HelloApplicationRunner...run...");
    }
}
```

```java
@Component
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("HelloCommandLineRunner...run...");
    }
}
```



最终执行的顺序是

```properties
HelloSpringApplicationRunLister...starting...
HelloSpringApplicationRunLister...starting...
HelloSpringApplicationRunLister...environmentPrepared...
HelloApplicationContextInitializer...initialize...
HelloSpringApplicationRunLister...contextPrepared...
HelloSpringApplicationRunLister...contextLoaded...
HelloSpringApplicationRunLister...finished...
HelloSpringApplicationRunLister...environmentPrepared...
HelloApplicationContextInitializer...initialize...
HelloSpringApplicationRunLister...contextPrepared...
HelloSpringApplicationRunLister...contextLoaded...
HelloApplicationRunner...run...
HelloCommandLineRunner...run...
HelloSpringApplicationRunLister...finished...
```

