# SpringBoot笔记



## 构建SpringBoot项目

SpringBoot不用像Spring那样需要在构建项目初期写很多配置文件，针对很多Spring应用程序常见的功能，SpringBoot自动提供相关的配置，简化开发流程，下面我们简单介绍一些SpringBoot使用和遇到的问题。

### 添加Maven依赖

程序是采用Maven构建的，所以我们首先要添加SpringBoot的相关依赖。进入Spring官网，官网教程给出的依赖如下：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
<dependency>
    <groupId>org.springframework.boot</groupId>    
        <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

上面依赖就是SpringBoot构建项目时所需要的Jar包依赖，SpringBoot在设计的宗旨就是简化开发流程，因此，Maven的相关依赖相比于我们平时构建的Spring程序也简单了很多，SpringBoot在底层都进行了相关的封装。上面依赖中，其中spring-boot-starter-parent是父项目，而spring-boot-starter-web则是web所用到的依赖，spring-boot-starter-test是测试相关的依赖。



### 项目的目录结构

SpringBoot本质上就是Spring项目，所以它的项目目录结构没有其他的变化，只是在src/main/java下会有一个启动类，用于启动项目，这里我们把Application.java放在了com.cqupt.sleep下。

![](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/springboot_1.png)

从上图我们可以看到项目的结构，目录com.cqupt.sleep结构如下：

```
com
	+- cqupt
		+- sleep
			+- Application.java
			+- controller
			+- service
			+- dao
			+- po
```

和传统的Spring项目没有太大的本质区别，只是在com.cqupt.sleep下多了个启动类Application.java，这个类用于启动整个SpringBoot项目。从这里我们就可以看出SpringBoot的方便之处，不用将项目部署到容器中，只需要运行Application.java类即可执行项目。



### 配置类Application.java

```java
package com.cqupt.sleep;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author weigs
 * @date 2018/5/28 0028
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

上面就是一个简单的配置类，从开头我们可以看到类的注解@SpringBootApplication。查看该注解的源代码，发现@SpringBootApplication 被 @SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan 注解所修饰。 

- @SpringBootConfiguration：就是Spring注解体系中`@Configuration`注解的替代品，`@SpringBootConfiguration`只是`@Configuration`的一个壳
- @EnableAutoConfiguration： 顾名思义，`@EnableAutoConguration`是自动化配置。那就是`@Configuration`的高级形式，其本质不变的就是，最终形成的配置放在 beans内部，和`@Bean`的效果相同。不过它的适用范围大部分是内部默认包。也就是它对这些Bean的有特殊要求 
- @ComponentScan ：通过该注解，实现了SpringBoot默认启动类扫描同包以及子包下的注解，所以在Application.class上引入该注解，就会扫描所在包以及子包的注解



### 配置文件

虽然说SpringBoot简化了很多配置，但是一些项目还是需要在配置文件中进行配置，SpringBoot项目使用一个全局的配置文件application.properties或者是application.yml，一般都在src/main/resources下。 两种形式的文件都能使用，不过我一般使用application.yml，yml文件看起来节奏感比较强，方便阅读。

```yml
server:
  port: 9090
  tomcat:
    max-connections: 200
    max-threads: 200
    uri-encoding: utf-8
```

上面是一个配置文件的简单介绍，通过这个配置文件我们可以发现，设置tomcat启动端口为9090，最大连接数为200等，从这里我们也可以看出yml文件的可阅读性很强。不仅tomcat的配置在这里更改，后面我们使用连接池以及整合redis等配置信息都会在这个全局的配置文件中进行更改。



### 整合Mybatis

在Maven中添加Mybatis相关的依赖，具体如下：

```xml

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
		
```

然后就是配置数据源，我们采用druid连接池作为数据源，具体在application.yml中的配置如下：

```
spring:
	datasource:
        url: jdbc:mysql://127.0.0.1:3306/depot
        username: root
        password: root
        # 使用druid数据源
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.jdbc.Driver
        filters: stat
        maxActive: 20
        initialSize: 1
        maxWait: 60000
        minIdle: 1
        timeBetweenEvictionRunsMillis: 60000
        minEvictableIdleTimeMillis: 300000
        validationQuery: select 'x'
        testWhileIdle: true
        testOnBorrow: false
        testOnReturn: false
        poolPreparedStatements: true
        maxOpenPreparedStatements: 20
```

然后通过配置类去读取配置文件中的信息，在项目目录中新建一个config文件夹，里面用于存放项目的相关配置类，druid的相关配置类如下：

```java
package com.cqupt.sleep.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.sql.SQLException;

/**
 * @author weigs
 * @date 2018/5/29 0029
 */
@Configuration
@ConfigurationProperties(prefix = "spring.datasources")
public class DatasourceConfig {
    private static final Logger LOG = LoggerFactory.getLogger(DatasourceConfig.class);
    private String type;
    private String url;
    private String driverClassName;
    private String username;
    private String password;
    private Integer initialSize;
    private Integer minIdle;
    private Integer maxActive;
    private Integer maxWait;
    private Integer timeBetweenEvictionRunsMillis;
    private Integer minEvictableIdleTimeMillis;
    private String validationQuery;
    private Boolean testWhileIdle;
    private Boolean testOnBorrow;
    private Boolean testOnReturn;
    private Boolean poolPreparedStatements;
    private Integer maxOpenPreparedStatements;
    private String filters;

    @Bean
    public ServletRegistrationBean druidServlet() {
        ServletRegistrationBean registrationBean = new ServletRegistrationBean();
        registrationBean.setServlet(new StatViewServlet());
        registrationBean.addUrlMappings("/druid/*");
        registrationBean.addInitParameter("loginUsername", username);
        registrationBean.addInitParameter("loginPassword", password);
        return registrationBean;
    }

    @Bean
    public DataSource druidDataSource() {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(url);
        druidDataSource.setUsername(username);
        druidDataSource.setPassword(password);
        druidDataSource.setDriverClassName(driverClassName);
        druidDataSource.setInitialSize(initialSize);
        druidDataSource.setMinIdle(minIdle);
        druidDataSource.setMaxActive(maxActive);
        druidDataSource.setMaxWait(maxWait);
        druidDataSource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
        druidDataSource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        druidDataSource.setValidationQuery(validationQuery);
        druidDataSource.setTestWhileIdle(testWhileIdle);
        druidDataSource.setTestOnBorrow(testOnBorrow);
        druidDataSource.setTestOnReturn(testOnReturn);
        try {
            druidDataSource.setFilters(filters);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return druidDataSource;
    }

  //省略getter和setter方法

}

```

最后就是在application.yml中配置Mybatis相关的配置信息，主要是指定Mybatis的配置文件和对应的实体类所在的位置：

```
mybatis:
  mapper-locations: classpath:mapping/*.xml
  type-aliases-package: com.cqupt.sleep.po
```

**在启动的时候出现错误，具体错误内容是`service`层找不到`dao`中的对象，最后的解决方法是在启动类中添加对`dao`层的mapper扫描，加上如下代码就可以启动项目：**

```java
@SpringBootApplication
@MapperScan("com.cqupt.sleep.dao")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```



#### 整合PageHelper

一般来说项目中都有分页查询，Mybatis的分页支持不好，所以我们可以选择插件`PageHelper`来进行分页处理，SpringBoot整合Pagehelper具体如下：首先就是添加Maven依赖

```xml
		<dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.3</version>
        </dependency>
```

在我自己的测试中，可以直接使用默认的PageHelper的配置，能够直接运行。网上很多人都在配置文件中加了以下配置

```
pagehelper:
  helper-dialect: mysql
  reasonable: true
  support-methods-arguments: true
```

