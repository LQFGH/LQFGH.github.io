---
layout:     post
title:      "springboot - web开发"
subtitle:   "web开发技术栈"
date:       2019-08-11 11:10
author:     "LQFGH"
header-img: "img/spring-cloud-bg.png"
header-mask: 0.3
catalog:    true
tags:

  - springboot

---


# Ⅰ 简介



**`spring boot`** **自动配置原理**

我们发现 `spring boot` 什么都帮我们配置好了，但是它是怎么做到的呢。

```
  XXXAutoConfig：使用配置类自动配置
  XXXProperties：封装配置信息
```

![1565460282633](/img/in-post/1565460282633.png)

> 1、其实 `spring boot` 是基于 `spring` 注解版进行进一步封装的

> 2、`spring boot` 使用`xxxProperties` 类对配置信息进行封装，并在 `xxxAutoConfig` 配置类中使用，这就是 `spring boot` 的精华所在。



# Ⅱ `spring boot ` 对静态资源的映射

> `spring boot` 关于资源映射的配置封装在 `ResourceProperties` 配置类中

`ResourceProperties`

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)public class ResourceProperties implements ResourceLoaderAware {
```



> `WebMvcAuotConfiguration` 是 `spring boot` 关于 `springmvc` 的自动配置，其实了解 `spring`  注解版的朋友知道其实就继承了 `spring` 的 `WebMvcConfigurerAdapter` 对 `springmvc` 进行配置的。

`WebMvcAutoConfiguration`

```java
		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Integer cachePeriod = this.resourceProperties.getCachePeriod();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				//所有webjars引入的资源可以用 /webjars/*来访问
                customizeResourceHandlerRegistration(
						registry.addResourceHandler("/webjars/**")
								.addResourceLocations(
										"classpath:/META-INF/resources/webjars/")
						.setCachePeriod(cachePeriod));
			}
            //添加静态资源映射，所有的静态资源到
            "classpath:/META-INF/resources/", 													"classpath:/resources/",
			"classpath:/static/", 
            "classpath:/public/" 
            "/"
            //等文件夹下找
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(
										this.resourceProperties.getStaticLocations())
						.setCachePeriod(cachePeriod));
			}
		}

		// 添加欢迎页为任意资源文件夹下的index.html
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(
				ResourceProperties resourceProperties) {
			return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
		}

		// 配置图标为任意资源文件夹下的favicon.ico图片
		@Configuration
		@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
		public static class FaviconConfiguration {

			private final ResourceProperties resourceProperties;

			public FaviconConfiguration(ResourceProperties resourceProperties) {
				this.resourceProperties = resourceProperties;
			}

			@Bean
			public SimpleUrlHandlerMapping faviconHandlerMapping() {
				SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
				mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
				mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
						faviconRequestHandler()));
				return mapping;
			}

			@Bean
			public ResourceHttpRequestHandler faviconRequestHandler() {
				ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
				requestHandler
						.setLocations(this.resourceProperties.getFaviconLocations());
				return requestHandler;
			}

		}	
```



#### 1）引入`webjars` 资源

**所有的 `webjars` 资源都到 `classpath:/META-INF/resources/webjars/ ` 找对应资源**

```xml
<dependency>    
	<groupId>org.webjars</groupId>    
	<artifactId>jquery</artifactId>    
<version>3.4.1</version></dependency>
```

![1565464227915](/img/in-post/1565464227915.png)

> 资源引入地址 ：http://localhost:8002/webjars/jquery/3.4.1/jquery.js

> `webjars` 官网地址：[http://www.webjars.org/](http://www.webjars.org/)



#### 2）静态资源地址

**所有的静态资源都会到静态资源**

```
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
"/"：当前项目的根路径
```

`localhost:8002/login.html` 会到静态资源文件夹中找 login.`html`



#### 3）欢迎页

**欢迎页会到所有资源文件夹中找 `index.html` 页面**



#### 4 ）`favicon` 图标

**图标会到所有资源文件夹中找 `favicon.ico ` 图片**



# Ⅲ 模板引擎





# Ⅳ `springmvc` 配置







# Ⅴ嵌入式 `servlet` 容器



