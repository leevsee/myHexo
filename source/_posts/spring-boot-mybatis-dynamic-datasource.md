---
title: spring boot mybatis dynamic datasource
date: 2018-07-30 00:30:27
tags:
---

最近，看了一下多数据源相关的知识，当成知识的积累，本篇是基于spring boot 2 + mybatis。  

### 原理

有几个数据源，就配置几套jdbc地址，分别建立连接池。然后在spring中，可以为每个数据源连接设置key，需要的时候，就通过key来获取相应的数据源连接即可。  
那然后怎么知道什么时候用哪个数据源呢，这时候就要用aop来判断具体方法、且需和当前线程进行绑定，如果是当前用户，且是某个方法，就使用该数据源，否则就是默认或者其他数据源。  

好啦，下面开始双数据源的例子吧。 
本栗子是两个的mysql数据库（因为懒得安装oracle。。。）。一个mysql，一个oracle也是同理。

### 1. 添加依赖

``` 
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.2</version>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
	<groupId>com.oracle</groupId>
	<artifactId>ojdbc6</artifactId>
	<version>12.1.0.1-atlassian-hosted</version>
</dependency>
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid-spring-boot-starter</artifactId>
	<version>1.1.10</version>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.46</version>
</dependency>
```


## 2. 添加注解类
这个注解类呢，是用于放在方法上面可以放在mapper和service类上面，用来判断使用哪个数据源。

``` 
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DS {
    DataSourceType dtaSourceType() default DataSourceType.DB_MASTER;
}
```

``` 
public enum DataSourceType {
    /** 主库 */
    DB_MASTER,
    /** 副库 */
    DB_SLAVE1
}
```


## 3. 读取多个数据源，并和调用线程绑定

``` 
@Configuration
public class DynamicDataSourceConfiguration {

    @Bean
    @ConfigurationProperties(prefix = "dynamic.datasource.master")
    public DataSource dbMaster() {
        return DruidDataSourceBuilder.create().build();
    }

    //TODO: 如有多个这里需要添加
    @Bean
    @ConfigurationProperties(prefix = "dynamic.datasource.slave1")
    public DataSource dbSlave1() {
        return DruidDataSourceBuilder.create().build();
    }

    /**
     * 核心动态数据源，添加多个数据源到spring中
     *
     * @return 数据源实例
     */
    @Bean
    public DataSource dynamicDataSource() {
        DynamicRoutingDataSource dataSource = new DynamicRoutingDataSource();
        dataSource.setDefaultTargetDataSource(dbMaster());
        Map<Object, Object> dataSourceMap = new HashMap<>(16);
        dataSourceMap.put(DataSourceType.DB_MASTER, dbMaster());
        dataSourceMap.put(DataSourceType.DB_SLAVE1, dbSlave1());
        //TODO: 如有多个这里需要添加
        dataSource.setTargetDataSources(dataSourceMap);
        return dataSource;
    }
	
	//...还有其他代码


}
```

线程绑定类:
``` 
public class DynamicDataSourceContextHolder {

    private static final ThreadLocal<DataSourceType> CURRENT_DATASOURCE = new ThreadLocal<>();

    /**
     * 清除当前数据源
     */
    public static void clear() {
        CURRENT_DATASOURCE.remove();
    }

    /**
     * 获取当前使用的数据源
     *
     */
    public static DataSourceType get() {
        return CURRENT_DATASOURCE.get();
    }

    /**
     * 设置当前使用的数据源
     *
     */
    public static void set(DataSourceType value) {
        CURRENT_DATASOURCE.set(value);
    }

}
```


## 4. spring多数据源和aop配置

重写`determineCurrentLookupKey()`方法，通过该方法取得所要的数据库。
```
public class DynamicRoutingDataSource extends AbstractRoutingDataSource {

    private static final Logger logger = LoggerFactory.getLogger(DynamicRoutingDataSource.class);

    @Override
    protected Object determineCurrentLookupKey() {
        logger.info("当前数据源："+ DynamicDataSourceContextHolder.get());
        return DynamicDataSourceContextHolder.get();
    }
}
```

通过`spring aop`，来拦截带`@DS`注解的方法，设置具体的数据源。

```
@Aspect
@Order(-1)
@Component
public class DynamicDataSourceAspect {

    private static final Logger logger = LoggerFactory.getLogger(DynamicDataSourceAspect.class);

    /**
     * 执行方法前更换数据源
     *
     */
    @Before("@annotation(ds)")
    public void doBefore(JoinPoint joinPoint, DS ds) {
        DataSourceType dataSourceType = ds.dtaSourceType();
        if (dataSourceType == DataSourceType.DB_SLAVE1) {
            logger.info("设置数据源为:" + DataSourceType.DB_SLAVE1);
            DynamicDataSourceContextHolder.set(DataSourceType.DB_SLAVE1);
        } else {
            logger.info("使用默认数据源:" + DataSourceType.DB_MASTER);
            DynamicDataSourceContextHolder.set(DataSourceType.DB_MASTER);
        }
    }

    /**
     * 执行方法后清除数据源设置
     *
     */
    @After("@annotation(ds)")
    public void doAfter(JoinPoint joinPoint, DS ds) {
        logger.info("当前数据源为："+ds.dtaSourceType()+"。然后执行清理数据源设置方法");
        DynamicDataSourceContextHolder.clear();
    }

}
```


## 5. 测试
完美通过！！！

![enter description here][1]

下次，带来个spring mybatis plus的多数据源配置。mybatis plus这个开源库还是不错的。

最后附上，具体代码地址：https://github.com/leevsee/spring-boot-dynamic-datasource



[1]: http://lixin.piaozu.com.cn/spring_mybatis_dynamic_datasource.png

