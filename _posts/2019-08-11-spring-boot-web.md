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



[TOC]



------



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



------



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



------



# Ⅲ 模板引擎 `thymeleaf` 使用



#### 1）导入 `thymeleaf` 的 `starter`

```xml
<dependency>    
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```



#### 2）修改 `thymeleaf` 的版本

- `thyeamleaf` 版本可以在 `GitHub` 上看到

![1565512481604](/img/in-post/1565512481604.png)



- 导入 `spring-boot-starter-thymeleaf` 依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```



- `spring boot` 官网文档修改 `thyeamleaf` 版本

![1565512552682](/img/in-post/1565512552682.png)



- 修改 `thrmeleaf` 版本号

  ```xml
  <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
  <thymeleaf-layout-dialect.version>2.4.1</thymeleaf-layout-dialect.version>
  ```

> 我这里直接使用 `github` 上最新的版本

![1565513389955](/img/in-post/1565513389955.png)

> 需要注意的是 `thymeleaf` 3版本需要适配 `thymeleaf-layout` 2版本

#### 3）`thymeleaf` 默认配置规则



```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8"); 
    private static final MimeType DEFAULT_CONTENT_TYPE =
        MimeType.valueOf("text/html");
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
```

> 当 `springmvc` 返回视图名称时，`springmvc` 默认会到 `templates` 文件中找 xxx.html页面



------



# Ⅳ `springmvc` 配置原理

#### 1）`springmvc` 自动配置

`spring boot` [官方文档](https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc)对 `springmvc`  自动配置的说明

![1565516463485](/img/in-post/1565516463485.png)

> 想要添加相应的组件，只需要在容器中注入该组件即可

------



`springmvc` 自动配置类

```java
public class WebMvcAutoConfiguration
```

**配置类中有一个静态内部类**

```java
@Configuration@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {
```



> 熟悉 `spring` 注解驱动开发的朋友知道 `WebMvcConfigurerAdapter` 是注解驱动开发中配置 `spirngmvc` 需要继承的抽象类，而 `spring boot` 实际上也是这么做的



#### 2）扩展 `springmvc` 配置

​	以前在 xml 中的配置

```xml
<mvc:view-controller path="/success" view-name="success"/>
<mvc:interceptors>    
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>    	<mvc:interceptor>
        <mvc:mapping path="/employee/emps"/>
        <bean class="com.crud.interceptors.SecodnInteceptor"/>
    </mvc:interceptor>
    <bean class="com.crud.interceptors.FirstInteceptor"/>
</mvc:interceptors>
```

现在用 `spring boot` 可以这样配置，我们也继承 `WebMvcConfigurerAdapter` 抽象类，重写其中的方法

```java
@EnableWebMvcpublic class WebMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/success").setViewName("success");
    }
}
```

> 这里只配置一个资源映射



其实 `WebMvcConfigurerAdapter` 是实现了 `WebMvcConfigurer` 接口，后面会说到 `WebMvcConfigurer` 

```java
public abstract class WebMvcConfigurerAdapter implements WebMvcConfigurer {
```



那这里就有个问题了，为什么我们自己写的配置不会覆盖 `WebMvcAutoConfiguration` 中的自动配置

```java
@Configuration
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {
```

我们看看这里的 `@Import(EnableWebMvcConfiguration.class)` 注入容器中的组件是什么

```java
@Configuration
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
```

然后看 `DelegatingWebMvcConfiguration` 类，类中有这样一个方法

```java
@Autowired(required = false)
public void setConfigurers(List<WebMvcConfigurer> configurers) {
	if (!CollectionUtils.isEmpty(configurers)) {
    	this.configurers.addWebMvcConfigurers(configurers);
    }
}
```

首先从容器中注入 `List<WebMvcConfigurer>` ,是所有的 `WebMvcConfigurer` 配置类

然后我们随便找一个 `DelegatingWebMvcConfiguration`  类中的一个添加拦截器的方法，其他的添加组件的方法也是一样的

```java
@Overrideprotected 
void addInterceptors(InterceptorRegistry registry) {
    this.configurers.addInterceptors(registry);
}
```

然后点进去第三行看下是怎么添加拦截器的

```java
@Overridepublic
void addInterceptors(InterceptorRegistry registry) {
	for (WebMvcConfigurer delegate : this.delegates) {
    	delegate.addInterceptors(registry);
    }
}
```

------

再点进去第4行，是一个接口的抽象方法

![1565526818398](/img/in-post/1565526818398.png)

点进去图中的类中

```java
@Overridepublic
void addInterceptors(InterceptorRegistry registry) {
    for (WebMvcConfigurer delegate : this.delegates) {
        delegate.addInterceptors(registry);
    }
}
```

可以看出其实是拿到了所有的 `WebMvcConfigurer` ，然后调用添加拦截器方法



#### 3）全面接管 `springmvc`

由图中红色部分可以看出，若想要让自动配置全部失效，而使用我们自己的配置，则需要在配置类上添加 `@EnableWebMvc` 注解

![1565527323790](/img/in-post/1565527323790.png)

> 若想检验，可以根据 `spring boot` 给我们配置的 `webjars` 映射测试下，加 `@EnableWebMvc` 和不加看能否访问到 `webjars` 引入的资源



那为什么添加了 `@EnableWebMvc` 之后自动配置会失效呢？

```java
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {}
```

我们发现 `@EnableWebMvc`  导入了 `DelegatingWebMvcConfiguration` 

再看下自动配置类

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,      WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,      ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

发现 `@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)` ，就是说当容器中如果有了 `WebMvcConfigurationSupport` 类，则自动配置不会生效，`WebMvcConfigurationSupport` 是 `springmvc` 最基本的类，里面封装了基本的配置信息。



# Ⅴ嵌入式 `servlet` 容器

`spring boot` 默认为我们引入的是 `tomcat` 作为嵌入式 `servlet` 容器

![1565532365311](/img/in-post/1565532365311.png)



使用配置文件配置 `server`

```yml
server:
	port: 8080
	context-path: spring-boot
    tomcat:
    	max-connections: 10
    	uri-encoding: utf-8
```



#### 1）`servlet` 容器自动配置原理

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties
    implements EmbeddedServletContainerCustomizer, EnvironmentAware, Ordered {
```

`ServerProperties` 实现了 `EmbeddedServletContainerCustomizer` 接口，并实现了 `customize` 方法

```java
@Overridepublic void customize(ConfigurableEmbeddedServletContainer container) {
    if (getPort() != null) {
        container.setPort(getPort());
    }
    if (getAddress() != null) {
        container.setAddress(getAddress());
    }
    if (getContextPath() != null) {
        container.setContextPath(getContextPath());
    }   if (getDisplayName() != null) {
        container.setDisplayName(getDisplayName());
    }   if (getSession().getTimeout() != null) {
        container.setSessionTimeout(getSession().getTimeout());
    }
    container.setPersistSession(getSession().isPersistent());
    container.setSessionStoreDir(getSession().getStoreDir());
    if (getSsl() != null) {
        container.setSsl(getSsl());
    }
    if (getJspServlet() != null) {
        container.setJspServlet(getJspServlet());
    }
    if (getCompression() != null) {
        container.setCompression(getCompression());
    }
    container.setServerHeader(getServerHeader());
    if (container instanceof TomcatEmbeddedServletContainerFactory) {
        getTomcat().customizeTomcat(this,
                                    (TomcatEmbeddedServletContainerFactory)
                                    container);
    }
    if (container instanceof JettyEmbeddedServletContainerFactory) {
        getJetty().customizeJetty(this,
                                  (JettyEmbeddedServletContainerFactory) container);
    }
    if (container instanceof UndertowEmbeddedServletContainerFactory) {
        getUndertow().customizeUndertow(this,
                                        (UndertowEmbeddedServletContainerFactory)
                                        container);
    }
    container.addInitializers(new SessionConfiguringInitializer(this.session));
    container.addInitializers(new InitParameterConfiguringServletContextInitializer(
        getContextParameters()));
}
```



#### 2）自定义 `servlet` 容器配置

自定义 `servet` 容器配置需要在 `ioc` 容器中添加一个 `EmbeddedServletContainerCustomizer` 实例

```java
@Bean
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return container -> {
        container.setPort(8082);
        container.setContextPath("/boot");
    };
}
```

> 自定义配置会覆盖配置文件中的配置



#### 3）注册 `servlet` 三大组件

先把三大组件各实现一个

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws
        ServletException, IOException {
        resp.getWriter().write("MyServlet'doGet...");
    }
}
```



```java
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("MyListener'contextInitialized...");
    }
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("MyListener'contextDestroyed...");
    }
}
```



```java
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse
      servletResponse, FilterChain filterChain) throws IOException,ServletException {
       
        System.out.println("MyFilter'doFilter...");
        filterChain.doFilter(servletRequest,servletResponse);
    }
    @Override
    public void destroy() {}
}
```

<br/>

在配置类中注入

```java
@Bean
public ServletListenerRegistrationBean servletListenerRegistrationBean(){
    return new ServletListenerRegistrationBean<>(new MyListener());
}
@Bean
public FilterRegistrationBean filterRegistrationBean(){
    FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
    filterRegistrationBean.setFilter(new MyFilter());
    filterRegistrationBean.setUrlPatterns(Collections.singletonList("/*"));
    return filterRegistrationBean;
}
@Bean
public ServletRegistrationBean servletRegistrationBean(){
    ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new MyServlet(), "/myServlet");
    servletRegistrationBean.setLoadOnStartup(Integer.MAX_VALUE);
    return servletRegistrationBean;
}
@Bean
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return container -> {
        container.setPort(8082);
        container.setContextPath("/boot");
    };
}
```

> 每种组件对应一种 `xxxRegistrationBean` 类，只需要在 `IOC` 容器中添加组件对应的  `RegistrationBean` 即可



`spring boot`注入组件的例子

```java
@Bean(name =
      DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name =
                   DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration(
    DispatcherServlet dispatcherServlet) {
    ServletRegistrationBean registration = new ServletRegistrationBean(
        dispatcherServlet, this.serverProperties.getServletMapping());
    registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
    registration.setLoadOnStartup(
        this.webMvcProperties.getServlet().getLoadOnStartup());
    if (this.multipartConfig != null) {
        registration.setMultipartConfig(this.multipartConfig);
    }
    return registration;
}
```

> 这是 `spring boot` 注入 `DispatcherServlet` 的代码



#### 4）`servlet` 容器的切换

`spring boot` 支持 `Tomcat` `Jetty` `Undertow` 三个`servlet` 容器

![1565539251549](/img/in-post/1565539251549.png)

- jetty：长连接引用，如聊天
- undertow：高并发



其实切换`servlet` 容器很简单，只需要依赖排除掉 `spring-boot-starter-web` 中默认的`tomcat` 容器，然后引入其他的 `servlet` 容器即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions></dependency><dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```



#### 5）嵌入式 `servlet` 自动配置原理

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
public class EmbeddedServletContainerAutoConfiguration {
    
	/**
	 * Nested configuration if Tomcat is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search
                              = SearchStrategy.CURRENT)
	public static class EmbeddedTomcat {

		@Bean
		public TomcatEmbeddedServletContainerFactory
            tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}

	}

	/**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class,
			WebAppContext.class })// 当前有相关的依赖
    //容器中没有注入 EmbeddedServletContainerFactory 嵌入式容器工厂
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search
                              = SearchStrategy.CURRENT)
	public static class EmbeddedJetty {

		@Bean
		public JettyEmbeddedServletContainerFactory 
            jettyEmbeddedServletContainerFactory() {
			return new JettyEmbeddedServletContainerFactory();
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search
                              = SearchStrategy.CURRENT)
	public static class EmbeddedUndertow {

		@Bean
		public UndertowEmbeddedServletContainerFactory 
            undertowEmbeddedServletContainerFactory() {
			return new UndertowEmbeddedServletContainerFactory();
		}

	}

	/**
	 * Registers a {@link EmbeddedServletContainerCustomizerBeanPostProcessor}.
     Registered
	 * via {@link ImportBeanDefinitionRegistrar} for early registration.
	 */
	public static class BeanPostProcessorsRegistrar
			implements ImportBeanDefinitionRegistrar, BeanFactoryAware {

		private ConfigurableListableBeanFactory beanFactory;

		@Override
		public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
			if (beanFactory instanceof ConfigurableListableBeanFactory) {
				this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
			}
		}

		@Override
		public void registerBeanDefinitions(AnnotationMetadata
                                            importingClassMetadata,
				BeanDefinitionRegistry registry) {
			if (this.beanFactory == null) {
				return;
			}
			registerSyntheticBeanIfMissing(registry,
					"embeddedServletContainerCustomizerBeanPostProcessor",
					EmbeddedServletContainerCustomizerBeanPostProcessor.class);
			registerSyntheticBeanIfMissing(registry,
					"errorPageRegistrarBeanPostProcessor",
					ErrorPageRegistrarBeanPostProcessor.class);
		}

		private void registerSyntheticBeanIfMissing(BeanDefinitionRegistry registry,
				String name, Class<?> beanClass) {
			if (ObjectUtils.isEmpty(
					this.beanFactory.getBeanNamesForType(beanClass, true, false))) {
				RootBeanDefinition beanDefinition = new 
                    RootBeanDefinition(beanClass);
				beanDefinition.setSynthetic(true);
				registry.registerBeanDefinition(name, beanDefinition);
			}
		}

	}

}

```

> 第 12 、31、48 行当容器中没有没有 `EmbeddedServletContainerFactory` 组件才会注入相应的 `servet ` 容器.
>
> 第11、 30、48 行只有引入了响应容器的依赖才会导入对应的容器



容器工厂的作用

```java
public interface EmbeddedServletContainerFactory {
 
    EmbeddedServletContainer getEmbeddedServletContainer(
        ServletContextInitializer... initializers);
}
```

> 只有一个方法,就是获取 `servlet` 容器

![1565541029115](/img/in-post/1565541029115.png)

我们就以 `tomcat` 对 `TomcatEmbeddedServletContainerFactory`  接口的实现`TomcatEmbeddedServletContainerFactory` 为例

![1565622344209](/img/in-post/1565622344209.png)

> `TomcatEmbeddedServletContainerFactory` 实现了 `getEmbeddedServletContainer` 的抽象方法，设置了 `Tomcat` 中的属性，并返回。



看下 `getTomcatEmbeddedServletContainer` 方法

```java
protected TomcatEmbeddedServletContainer getTomcatEmbeddedServletContainer(
    Tomcat tomcat) {
    return new TomcatEmbeddedServletContainer(tomcat, getPort() >= 0);
}
```

> 这里直接 创建了 `TomcatEmbeddedServletContainer`  实例，看看具体是怎么创建的

![1565622722605](/img/in-post/1565622722605.png)



> 创建容器时调用了 `initialize` 方法，在 `initialize` 方法中调用了 `tomcat.start()` 启动 `tomcat`



那我们对容器的配置是怎么生效的呢？

- 配置 `ServerProperties`

```yml
server:
	port: 8080
    context-path: /spring-boot
```

- 实现 `EmbeddedServletContainerCustomizer` 并将其添加到容器中 

> 上面 2）中有相关代码，这里就不贴出来了



配置生效原理解析

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
public class EmbeddedServletContainerAutoConfiguration {
```

`EmbeddedServletContainerAutoConfiguration` 自动配置类的 `@Import(BeanPostProcessorsRegistrar.class)` 导入了一个组件 `BeanPostProcessorsRegistrar`，其实就是 `EmbeddedServletContainerAutoConfiguration` 的静态内部类， 接下来我们看下这个类是个什么东西

```java
public static class BeanPostProcessorsRegistrar
    implements ImportBeanDefinitionRegistrar, BeanFactoryAware {
    private ConfigurableListableBeanFactory beanFactory;
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        if (beanFactory instanceof ConfigurableListableBeanFactory) {
            this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
        }
    }
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
                                        BeanDefinitionRegistry registry) {
        if (this.beanFactory == null) {
            return;
        }
        registerSyntheticBeanIfMissing(registry,
                               "embeddedServletContainerCustomizerBeanPostProcessor",
						EmbeddedServletContainerCustomizerBeanPostProcessor.class);
        registerSyntheticBeanIfMissing(registry,
                                       "errorPageRegistrarBeanPostProcessor",
                                       ErrorPageRegistrarBeanPostProcessor.class);
    }
    private void registerSyntheticBeanIfMissing(BeanDefinitionRegistry registry,
                                                String name, Class<?> beanClass) {
        if (ObjectUtils.isEmpty(this.beanFactory.getBeanNamesForType(beanClass, true,
                                                                     false))) {
            RootBeanDefinition beanDefinition = new RootBeanDefinition(beanClass);
            beanDefinition.setSynthetic(true);
            registry.registerBeanDefinition(name, beanDefinition);
        }
    }
}
```

`BeanPostProcessorsRegistrar` 实现了 `ImportBeanDefinitionRegistrar` ，`ImportBeanDefinitionRegistrar` 是`spring` 中很重要的组件 ，实现其 `registerBeanDefinitions` 方法用于在 `IOC` 容器中注入组件，可以看到实现的方法中注入了 `EmbeddedServletContainerCustomizerBeanPostProcessor` ，那这个类又是干什么的呢？

```java
public class EmbeddedServletContainerCustomizerBeanPostProcessor
    implements BeanPostProcessor, BeanFactoryAware {
```

`EmbeddedServletContainerCustomizerBeanPostProcessor` 实现了 `BeanPostProcessor` 接口，这个接口是

`spring` 中非常重要的一个接口，该接口又两个方法，`postProcessBeforeInitialization` 和 `postProcessAfterInitialization`，分别在 `bean` 被创建了，但是没有赋初始值的前后被调用，接下来看看 `postProcessBeforeInitialization` 方法中做了什么。

```java
@Overridepublic Object postProcessBeforeInitialization(Object bean, String beanName)
    throws BeansException {
    if (bean instanceof ConfigurableEmbeddedServletContainer) {
        postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
    }
    return bean;
}
```

该方法中调用了所有的 同类中的 `postProcessBeforeInitialization` ，接下来看该方法

```java
private void postProcessBeforeInitialization(
    ConfigurableEmbeddedServletContainer bean) {
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}
```

在该方法中调用了便利调用了所有 `EmbeddedServletContainerCustomizer` 类型的组件的 `customize` 方法，而对于自动配置和我们自定义配置 `servlet` 类型的组件都是 `EmbeddedServletContainerCustomizer`  类型的

**自定义配置**

```java
@Bean
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return container -> {
        container.setPort(8082);
        container.setContextPath("/boot");
    };
}
```

**自动配置**

```java
private static class DuplicateServerPropertiesDetector implements
    EmbeddedServletContainerCustomizer, Ordered, ApplicationContextAware {
```

> 不管是自动配置还是自定义配置都是 `EmbeddedServletContainerCustomizer` 类型的组件，并且都实现了 `customize` 方法，所以自动配置和自定义配置都会生效，只不过自动配置设置的优先级要低于自定义配置的优先级，所以自定义配置的属性会覆盖自动配置的属性，而自定义配置没有配置的属性会使用自动配置的属性



#### 6）嵌入式 `servlet` 启动原理

上面说到了三大容器都实现了 `EmbeddedServletContainerFactory` 接口，实现了 它的抽象方法 `getEmbeddedServletContainer` ，在该方法中实现了各容器的创建逻辑，接下来我们看看 `EmbeddedServletContainerFactory` ` 接口的方法是在哪里被调用来创建容器的。

`AbstractApplicationContext.class`

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization
                            - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

在 `AbstractApplicationContext` 类中的 `refresh` 方法，是 `spring` 包含了整个生命周期的方法，其中  30 行处调用 `onRefresh` 方法

```java
@Override
protected void onRefresh() {
    super.onRefresh();
    try {
        createEmbeddedServletContainer();
    }
    catch (Throwable ex) {
        throw new ApplicationContextException("Unable to start embedded
                                              container",ex);
	}
}
```

我们看第 5 行 `createEmbeddedServletContainer` 好像是创建 `servlet` 容器的意思，我们进去看看

```java
private void createEmbeddedServletContainer() {
    EmbeddedServletContainer localContainer = this.embeddedServletContainer;
    ServletContext localServletContext = getServletContext();
    if (localContainer == null && localServletContext == null) {
        EmbeddedServletContainerFactory containerFactory =
            getEmbeddedServletContainerFactory();
        this.embeddedServletContainer = containerFactory
            .getEmbeddedServletContainer(getSelfInitializer());
    }
    else if (localServletContext != null) {
        try {
            getSelfInitializer().onStartup(localServletContext);
        }
        catch (ServletException ex) {
            throw new ApplicationContextException("Cannot initialize servlet
                                                  context",
                                                  ex);
        }
    }
    initPropertySources();
}

```

- 第 5 行创建  `EmbeddedServletContainerFactory`  容器工厂

- 容器看到要创建  `EmbeddedServletContainerFactory`  容器工厂,会获取所有的订制器来处理 `servlet` 配置

- 第 7 行代码用 `EmbeddedServletContainerFactory` 调用了 `getEmbeddedServletContainer` 方法，至此 `servletContrainer` 容器创建成功。

#### 7）使用外置 `servlet` 容器

嵌入式 `servlet` 容器优点：

- 优点：简单、便捷
- 缺点：默认不支持 `jsp` 、优化定制比较复杂

外置 `servlet` 容器：外面安装类似 `Tomcat` --将应用打成 `war` 包，部署在 `servlet` 容器中

**步骤**

- 创建一个 `war` 项目，搭建好 `web` 项目的目录机构
- 将嵌入式 `Tomcat` 容器指定为 `provide`

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```

- 必须编写一个**SpringBootServletInitializer**的子类，并调用主配置类

```java
public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(WarJspApplication.class);
    }
}
```

```java
@SpringBootApplication
public class WarJspApplication {

    public static void main(String[] args) {
        SpringApplication.run(WarJspApplication.class,args);
    }
}
```

