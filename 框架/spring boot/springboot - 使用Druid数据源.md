# Druid介绍
Druid是一个JDBC组件，druid 是阿里开源在 github 上面的数据库连接池,它包括三部分： 
* DruidDriver 代理Driver，能够提供基于Filter－Chain模式的插件体系。 
* DruidDataSource 高效可管理的数据库连接池。 
* SQLParser 专门解析 sql 语句


# 引入Druid

```
        <dependency>
		    <groupId>com.alibaba</groupId>
		    <artifactId>druid-spring-boot-starter</artifactId>
		    <version>1.1.10</version>
		</dependency>
```

# 配置文件

```
# 驱动配置信息
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/test
spring.datasource.username = root
spring.datasource.password = root
spring.datasource.driverClassName = com.mysql.jdbc.Driver

# 连接池的配置信息
# 初始化大小，最小，最大
spring.datasource.initialSize=5
spring.datasource.minIdle=5
spring.datasource.maxActive=20
# 配置获取连接等待超时的时间
spring.datasource.maxWait=60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.timeBetweenEvictionRunsMillis=60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.minEvictableIdleTimeMillis=300000
spring.datasource.validationQuery=SELECT 1 FROM DUAL
spring.datasource.testWhileIdle=true
spring.datasource.testOnBorrow=false
spring.datasource.testOnReturn=false
# 打开PSCache，并且指定每个连接上PSCache的大小
spring.datasource.poolPreparedStatements=true
spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
spring.datasource.filters=stat,wall,slf4j
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

#jpa设置
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

# 配置Druid数据源

```
package com.jx.config;

import java.sql.SQLException;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.PropertySource;

import com.alibaba.druid.pool.DruidDataSource;

@Configuration
public class DruidJDBConfig {

    @Value("${spring.datasource.url}")
    private String dbUrl;

    @Value("${spring.datasource.username}")
    private String username;

    @Value("${spring.datasource.password}")
    private String password;

    @Value("${spring.datasource.driverClassName}")
    private String driverClassName;

    @Value("${spring.datasource.initialSize}")
    private int initialSize;

    @Value("${spring.datasource.minIdle}")
    private int minIdle;

    @Value("${spring.datasource.maxActive}")
    private int maxActive;

    @Value("${spring.datasource.maxWait}")
    private int maxWait;

    @Value("${spring.datasource.timeBetweenEvictionRunsMillis}")
    private int timeBetweenEvictionRunsMillis;

    @Value("${spring.datasource.minEvictableIdleTimeMillis}")
    private int minEvictableIdleTimeMillis;

    @Value("${spring.datasource.validationQuery}")
    private String validationQuery;

    @Value("${spring.datasource.testWhileIdle}")
    private boolean testWhileIdle;

    @Value("${spring.datasource.testOnBorrow}")
    private boolean testOnBorrow;

    @Value("${spring.datasource.testOnReturn}")
    private boolean testOnReturn;

    @Value("${spring.datasource.poolPreparedStatements}")
    private boolean poolPreparedStatements;

    @Value("${spring.datasource.maxPoolPreparedStatementPerConnectionSize}")
    private int maxPoolPreparedStatementPerConnectionSize;

    @Value("${spring.datasource.filters}")
    private String filters;

    @Value("{spring.datasource.connectionProperties}")
    private String connectionProperties;

    @Bean     //声明其为Bean实例
    @Primary  //在同样的DataSource中，首先使用被标注的DataSource
    public DataSource dataSource() {
        DruidDataSource datasource = new DruidDataSource();

        datasource.setUrl(this.dbUrl);
        datasource.setUsername(username);
        datasource.setPassword(password);
        datasource.setDriverClassName(driverClassName);

        //configuration
        datasource.setInitialSize(initialSize);
        datasource.setMinIdle(minIdle);
        datasource.setMaxActive(maxActive);
        datasource.setMaxWait(maxWait);
        datasource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
        datasource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        datasource.setValidationQuery(validationQuery);
        datasource.setTestWhileIdle(testWhileIdle);
        datasource.setTestOnBorrow(testOnBorrow);
        datasource.setTestOnReturn(testOnReturn);
        datasource.setPoolPreparedStatements(poolPreparedStatements);
        datasource.setMaxPoolPreparedStatementPerConnectionSize(maxPoolPreparedStatementPerConnectionSize);
        try {
            datasource.setFilters(filters);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        datasource.setConnectionProperties(connectionProperties);

        return datasource;
    }
}
```

# jpa配置

```

import java.util.Map;

import javax.persistence.EntityManager;
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactoryPrimary",  
        transactionManagerRef="transactionManagerPrimary",  
        basePackages= { "com.jx.dao"}) //设置Repository所在位置  
public class JpaConfig {
	
	@Autowired  
    @Qualifier("dataSouce")
    private DataSource dataSouce;
	
    @Bean(name = "entityManagerPrimary")  
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {  
        return entityManagerFactoryPrimary(builder).getObject().createEntityManager();  
    }  
    @Bean(name = "entityManagerFactoryPrimary")  
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryPrimary (EntityManagerFactoryBuilder builder) {  
        return builder  
                .dataSource(dataSouce)  
                .properties(getVendorProperties(dataSouce))  
                .packages("com.jx.bean") //设置实体类所在位置  
                .persistenceUnit("persistenceUnit")
                .build();
    }  
    @Autowired
    private JpaProperties jpaProperties;  
    
    private Map<String,String> getVendorProperties(DataSource dataSource) {
    	//如果数据源中存在不同类型的数据库，那么需要配置方言
    	jpaProperties.setDatabase(Database.MYSQL);
    	jpaProperties.getProperties().put("hibernate.dialect","org.hibernate.dialect.MySQL5Dialect");
        return jpaProperties.getHibernateProperties(dataSource);
    }
    
    @Bean(name = "transactionManagerPrimary")  
    public PlatformTransactionManager transactionManagerPrimary(EntityManagerFactoryBuilder builder) {  
        return new JpaTransactionManager(entityManagerFactoryPrimary(builder).getObject());  
    }  
}
```


# 引入Druid监控
通过 http://localhost:8080/druid/index.html 即可访问

```

import java.util.HashMap;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;

@Configuration
public class DruidConfiguration {

  private static final Logger log = LoggerFactory.getLogger(DruidConfiguration.class);

  @Bean
  public ServletRegistrationBean druidServlet() {
    log.info("init Druid Servlet Configuration ");
    ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean();
    servletRegistrationBean.setServlet(new StatViewServlet());
    servletRegistrationBean.addUrlMappings("/druid/*");
    Map<String, String> initParameters = new HashMap<String, String>();
    initParameters.put("loginUsername", "admin");// 用户名
    initParameters.put("loginPassword", "admin");// 密码
    initParameters.put("resetEnable", "false");// 禁用HTML页面上的“Reset All”功能
    initParameters.put("allow", ""); // IP白名单 (没有配置或者为空，则允许所有访问)
    //initParameters.put("deny", "192.168.20.38");// IP黑名单 (存在共同时，deny优先于allow)
    servletRegistrationBean.setInitParameters(initParameters);
    return servletRegistrationBean;
  }

  @Bean
  public FilterRegistrationBean filterRegistrationBean() {
    FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
    filterRegistrationBean.setFilter(new WebStatFilter());
    filterRegistrationBean.addUrlPatterns("/*");
    filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
    return filterRegistrationBean;
  }

}
```


# 码云地址
https://gitee.com/jiuxiao/springboot_druid.git