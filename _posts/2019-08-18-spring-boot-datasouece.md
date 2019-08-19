---
layout:     post
title:      "spring-boot-jdbc"
subtitle:   "spring boot连接数据库"
date:       2019-08-17 17:10
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:

  - springboot
---







# 

# Ⅰ`JDBC`

#### 一：添加依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```



###### 二：配置

```yml
server:
  port: 8080
spring:
  application:
    name: boot-jdbc
  datasource:
    username: root
    password: 1230
    url: jdbc:mysql://liqingfeng.com:3306/springboot?useSSL=false
    driver-class-name: com.mysql.jdbc.Driver

```

------



#### 二：自动配置原理

- 自动配置类：`DataSourceAutoConfiguration`

- `Properties` 类：`DataSourceProperties`
- `jdbctemplate`自动配置类：`JdbcTemplateAutoConfiguration`

```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
public class DataSourceAutoConfiguration {
    @Bean
	@ConditionalOnMissingBean
	public DataSourceInitializer dataSourceInitializer(DataSourceProperties
                                                       properties,
			ApplicationContext applicationContext) {
		return new DataSourceInitializer(properties, applicationContext);
	}
```

> 这里是数据源自动配置类注入容器的一些组件，接下来我们就看看这些组件都起到了什么作用



###### `DataSourceProperties`

```java
@EnableConfigurationProperties(DataSourceProperties.class)
```

导入配置属性类，就不多说了



###### `Registrar`

```java
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
```

```java
	static class Registrar implements ImportBeanDefinitionRegistrar {

		private static final String BEAN_NAME = "dataSourceInitializerPostProcessor";

		@Override
		public void registerBeanDefinitions(AnnotationMetadata
                                            importingClassMetadata,
				BeanDefinitionRegistry registry) {
			if (!registry.containsBeanDefinition(BEAN_NAME)) {
				GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
				beanDefinition.setBeanClass(DataSourceInitializerPostProcessor.class);
				beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
			// We don't need this one to be post processed otherwise it can cause a
			// cascade of bean instantiation that we would rather avoid.
				beanDefinition.setSynthetic(true);
				registry.registerBeanDefinition(BEAN_NAME, beanDefinition);
			}
		}

	}
```

`Registrar` 类实现了 `ImportBeanDefinitionRegistrar` ，实现了 `registerBeanDefinitions` 主要作用是在容器中添加 `DataSourceInitializerPostProcessor` 类

```java
class DataSourceInitializerPostProcessor implements BeanPostProcessor, Ordered {
    @Override
	public Object postProcessAfterInitialization(Object bean, String beanName)
			throws BeansException {
		if (bean instanceof DataSource) {
			// force initialization of this bean as soon as we see a DataSource
			this.beanFactory.getBean(DataSourceInitializer.class);
		}
		return bean;
	}
```

 `DataSourceInitializerPostProcessor` 类主要是把容器中的所有 `DataSource` 类型的组件强制初始化



###### `DataSourcePoolMetadataProvidersConfiguration`

```java
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
```

由 `DataSourcePoolMetadataProvidersConfiguration` 代码可以看出这个类主要是向容器中注入注入数据源，

由此也可以看出spring boot默认支持的连接池有 `org.apache.tomcat.jdbc.pool.DataSource` (默认)、`HikariDataSource`、`org.apache.commons.dbcp.BasicDataSource`、`org.apache.commons.dbcp2.BasicDataSource` 、自定义数据源 ，可以使用 `spring boot` 的 `spring.datasource.type` 配置修改数据源的类型

###### `DataSourceInitializer`

```java
 	@Bean
	@ConditionalOnMissingBean
	public DataSourceInitializer dataSourceInitializer(DataSourceProperties
                                                       properties,
			ApplicationContext applicationContext) {
		return new DataSourceInitializer(properties, applicationContext);
	}
```

```java
class DataSourceInitializer implements ApplicationListener<DataSourceInitializedEvent> {
    
    @PostConstruct
	public void init() {
		if (!this.properties.isInitialize()) {
			logger.debug("Initialization disabled (not running DDL scripts)");
			return;
		}
		if (this.applicationContext.getBeanNamesForType(DataSource.class, false,
				false).length > 0) {
			this.dataSource = this.applicationContext.getBean(DataSource.class);
		}
		if (this.dataSource == null) {
			logger.debug("No DataSource found so not initializing");
			return;
		}
		runSchemaScripts();
	}
    
    @Override
	public void onApplicationEvent(DataSourceInitializedEvent event) {
		if (!this.properties.isInitialize()) {
			logger.debug("Initialization disabled (not running data scripts)");
			return;
		}
		// NOTE the event can happen more than once and
		// the event datasource is not used here
		if (!this.initialized) {
			runDataScripts();
			this.initialized = true;
		}
	}
```

`DataSourceInitializer` 实现了 `ApplicationListener` 接口，实现了 `onApplicationEvent` 方法，该方法中

`runDataScripts()` 初始化所有 `classpath` 目录下 `data.sql` 、`data-all.sql`脚本和配置文件中配置的 `spring.datasource.data` 的脚本

该类中还有一个 `init` 方法，标注了 `@PostConstruct` 注解，该注解是 `JRS250` 规范的注解就相当于`spring`中创建bean时指定的 `init-method` 方法（构造器之后）

------



#### 三：`druid` 配置

###### 1）添加依赖

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
```



###### 2）属性配置

```yml
spring:
  application:
    name: boot-jdbc
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
  druid:
    url: jdbc:mysql://liqingfeng.com:3306/springboot?useSSL=false
    username: root
    password: 1230
    initial-size: 1
    min-idle: 1
    max-active: 5
    max-wait: 3000
    use-unfair-lock: true
    time-between-eviction-runs-millis: 10000
    min-evictable-idle-time-millis: 30000
    validation-query: select '1'
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
    use-global-data-source-stat: true
    max-pool-prepared-statement-per-connection-size: 10
    filters: stat,wall,slf4j
    pool-prepared-statements: true

```

> 这里的 `filters` 一定要配置，否则 `druid` 的 `SQL` 不能生效



###### 3）让配置的属性生效

```java
    @ConfigurationProperties(prefix = "spring.druid")
    @Bean
    public DruidDataSource druidDataSource(){
        return new DruidDataSource();
    }
```

> 主要是添加 `@ConfigurationProperties` 注解



4）配置 `druid` 监控

```java
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean servletRegistrationBean = new
            ServletRegistrationBean(new StatViewServlet(),"/druid/*");
        Map<String,String> initParams = new HashMap<String,String>(6);
        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","admin");
        initParams.put("allow","");
        servletRegistrationBean.setInitParameters(initParams);
        return servletRegistrationBean;
    }
    
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean filterRegistrationBean = new
            FilterRegistrationBean(new WebStatFilter());
        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        filterRegistrationBean.addInitParameter("sessionStatEnable","true");
        filterRegistrationBean.addInitParameter("profileEnable","true");
        filterRegistrationBean.addInitParameter("principalCookieName","USER_COOKIE");
        						             filterRegistrationBean.addInitParameter("principalSessionName","USER_SESSION");
        filterRegistrationBean.addUrlPatterns("/*");
        return filterRegistrationBean;
    }
```

> 配置 `StatViewServlet` 和 `WebStatFilter`



------



#### 四：整合 `Mybatis`



###### 1）注解版配置

其实注解版什么都不需要配置，只需要在接口上添加 `@Mapper` 注解，然后写接口，在接口上加注解，并写对应的 `SQL` 语句就可以了

```java
import com.feng.boot.mybatis.bean.Department;
import org.apache.ibatis.annotations.*;

/**
 * @author Lee
 */
@Mapper
public interface DepartmentMapper {

    /**
     * 根据 ID 查询 Department
     * @param id
     * @return
     */
    @Select("select * from department d where d.id = #{id}")
    Department getDeptById(Integer id);

    /**
     * 根据 ID 删除 Department
     * @param dId
     * @return
     */
    @Delete("delete from department d where d.id = #{id}")
    int deleteDepartment(Integer dId);;

    /**
     * 插入 Department
     * @param department
     * @return
     */
    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) value (#{departmentName})")
    int insertDepartment(Department department);

    /**
     * 更新 departmentName
     * @param department
     * @return
     */
    @Update("update department d set d.departmentName = ${departmentName} where d.id
            = ${id}")
    int updateSelective(Department department);

}

```

```java
@Options(useGeneratedKeys = true,keyProperty = "id")
```

> 是为了返回插入数据库中的记录的主键