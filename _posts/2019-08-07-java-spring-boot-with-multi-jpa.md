---
layout: post
title: "Java Spring Boot JPA配置多个数据源"
subtitle: "Java Spring Boot JPA With Multi Redis Source"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
  - MySQL
---

之前在6月20日的博文`Java Spring Boot Mybatis配置多个MySQL源`这篇文章中总结了如何在spring boot 中使用mybatis连接多个MySQL源。今天接手了一个项目，采用的是JPA，也要配置多数据源，摸索了一下，发现和Mybatis的配制方法还不一样。特此总结一下。

## 1 准备工作
首先准备好依赖关系，编辑项目根目录下的pom.xml文件，主要检查jpa和mysql-connector是否正确配置并安装。另外，要注意`spring-boot-starter-parent`这个包的版本号。因为parent的版本之间变动比较大，如果版本号相差太多，大概率会出现不兼容的情况。实测parent的1.x和2.0和2.1这三个大版本之间在某些方法上是不兼容的。
```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <skipTests>true</skipTests>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- spring-boot-starter-data-jpa -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```

## 2 配置多个数据源的链接信息
在项目的resource目录中，找到项目配置文件，一般是properties结尾的文件，在里面找到默认的数据库配置项：
```config
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```
将上述默认的数据库配置项改为实际的多数据源配置：
```config
spring.datasource.parrot.url=jdbc:mysql://127.0.0.2:3306/parrot?useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.parrot.username=test
spring.datasource.parrot.password=123456
spring.datasource.parrot.driver-class-name=com.mysql.jdbc.Driver
#primary数据源jpa配置，部分配置仅在此处生效
spring.jpa.primary.show-sql=true
spring.jpa.primary.generate-ddl=true
spring.jpa.primary.hibernate.ddl-auto=update
spring.jpa.primary.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect


spring.datasource.metasys.url=jdbc:mysql://127.0.0.1:3306/metasys
spring.datasource.metasys.username=test
spring.datasource.metasys.password=test
spring.datasource.metasys.driver-class-name=com.mysql.jdbc.Driver
#second数据源jpa配置
spring.jpa.second.generate-ddl=true
spring.jpa.second.hibernate.ddl-auto=update
spring.jpa.second.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```
注意在`datasource`后面就是自定义的数据源字段了，这里分别定义的是parrot和metasys两个数据元。此时IDE可能会提醒“无法解析”，不用理会，并不影响。

## 3 定义第一数据源类
```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;


@Configuration
public class PrimaryDataSource {
    /**
     * 扫描spring.datasource.primary开头的配置信息
     *
     * @return 数据源配置信息
     */
    @Primary
    @Bean(name = "primaryDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.parrot")
    public DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    /**
     * 获取主库数据源对象
     *
     * @param properties 注入名为primaryDataSourceProperties的bean
     * @return 数据源对象
     */
    @Primary
    @Bean(name = "primaryDataSource")
    public DataSource dataSource(@Qualifier("primaryDataSourceProperties") DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }

    /**
     * 该方法仅在需要使用JdbcTemplate对象时选用
     *
     * @param dataSource 注入名为primaryDataSource的bean
     * @return 数据源JdbcTemplate对象
     */
    @Primary
    @Bean(name = "primaryJdbcTemplate")
    public JdbcTemplate jdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```
这里需要注意的是，basePackages填写的是要使用该数据源的dao的目录，在指定的目录下，使用的就是该数据源连接。一个使用JPA的dao的写法大概是这个样子：
```java
import com.vipshop.parrot.job.basicInfo.model.DeployIds;
import org.springframework.data.jpa.repository.JpaRepository;

public interface DeployIdsRepository extends JpaRepository<DeployIds, String> {
}
```


# 4 定义第一数据源JPA
```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateSettings;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        // repository包名
        basePackages = "com.vipshop.parrot.job.basicInfo.dao",
        // 实体管理bean名称
        entityManagerFactoryRef = "primaryEntityManagerFactory",
        // 事务管理bean名称
        transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryConfig {
    /**
     * 扫描spring.jpa.primary开头的配置信息
     *
     * @return jpa配置信息
     */
    @Primary
    @Bean(name = "primaryJpaProperties")
    @ConfigurationProperties(prefix = "spring.jpa.primary")
    public JpaProperties jpaProperties() {
        return new JpaProperties();
    }

    /**
     * 获取主库实体管理工厂对象
     *
     * @param primaryDataSource 注入名为primaryDataSource的数据源
     * @param jpaProperties     注入名为primaryJpaProperties的jpa配置信息
     * @param builder           注入EntityManagerFactoryBuilder
     * @return 实体管理工厂对象
     */
    @Primary
    @Bean(name = "primaryEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(@Qualifier("primaryDataSource") DataSource primaryDataSource
            , @Qualifier("primaryJpaProperties") JpaProperties jpaProperties, EntityManagerFactoryBuilder builder) {
        return builder
                // 设置数据源
                .dataSource(primaryDataSource)
                // 设置jpa配置
                .properties(jpaProperties.getProperties())
                // 设置hibernate配置
                .properties(jpaProperties.getHibernateProperties(new HibernateSettings()))
                // 设置实体包名
                .packages("com.vipshop.parrot.job.basicInfo.model")
                // 设置持久化单元名，用于@PersistenceContext注解获取EntityManager时指定数据源
                .persistenceUnit("primaryPersistenceUnit")
                .build();
    }

    /**
     * 获取实体管理对象
     *
     * @param factory 注入名为primaryEntityManagerFactory的bean
     * @return 实体管理对象
     */
    @Primary
    @Bean(name = "primaryEntityManager")
    public EntityManager entityManager(@Qualifier("primaryEntityManagerFactory") EntityManagerFactory factory) {
        return factory.createEntityManager();
    }

    /**
     * 获取主库事务管理对象
     *
     * @param factory 注入名为primaryEntityManagerFactory的bean
     * @return 事务管理对象
     */
    @Primary
    @Bean(name = "primaryTransactionManager")
    public PlatformTransactionManager transactionManager(@Qualifier("primaryEntityManagerFactory") EntityManagerFactory factory) {
        return new JpaTransactionManager(factory);
    }
}
```
这里需要注意的是.packages里面填写的是使用该数据源的Model。一般model就是该数据源中某张表的映射。本例中的model示例：
```java
import javax.persistence.*;

@Entity
@Table(name="deploy_ids")
public class DeployIds {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private String id;
    private String component;

    @Column(name = "deploy_ids")
    private String deployIds;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getComponent() {
        return component;
    }

    public void setComponent(String component) {
        this.component = component;
    }

    public String getDeployIds() {
        return deployIds;
    }

    public void setDeployIds(String deployIds) {
        this.deployIds = deployIds;
    }
}
```

## 5 定义第二数据源
同理的，定义第二个甚至继续定义更多的数据源
```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;

@Configuration
public class SecondDataSource {
    /**
     * 扫描spring.datasource.second开头的配置信息
     *
     * @return 数据源配置信息
     */
    @Bean(name = "secondDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.metasys")
    public DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    /**
     * 获取从库数据源对象
     *
     * @param properties 注入名为secondDataSourceProperties的bean
     * @return 数据源对象
     */
    @Bean(name = "secondDataSource")
    public DataSource dataSource(@Qualifier("secondDataSourceProperties") DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }

    /**
     * 该方法仅在需要使用JdbcTemplate对象时选用
     *
     * @param dataSource 注入名为secondDataSource的bean
     * @return 数据源JdbcTemplate对象
     */
    @Bean(name = "secondJdbcTemplate")
    public JdbcTemplate jdbcTemplate(@Qualifier("secondDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

## 6 定义第二数据源JPA
```java
package com.vipshop.parrot.job.configurer;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateSettings;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;


@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        // repository包名
        basePackages = "com.vipshop.parrot.job.offline.dao",
        // 实体管理bean名称
        entityManagerFactoryRef = "secondEntityManagerFactory",
        // 事务管理bean名称
        transactionManagerRef = "secondTransactionManager"
)
public class SecondConfig {
    /**
     * 扫描spring.jpa.second开头的配置信息
     *
     * @return jpa配置信息
     */
    @Bean(name = "secondJpaProperties")
    @ConfigurationProperties(prefix = "spring.jpa.second")
    public JpaProperties jpaProperties() {
        return new JpaProperties();
    }

    /**
     * 获取从库实体管理工厂对象
     *
     * @param secondDataSource 注入名为secondDataSource的数据源
     * @param jpaProperties    注入名为secondJpaProperties的jpa配置信息
     * @param builder          注入EntityManagerFactoryBuilder
     * @return 实体管理工厂对象
     */
    @Bean(name = "secondEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(@Qualifier("secondDataSource") DataSource secondDataSource
            , @Qualifier("secondJpaProperties") JpaProperties jpaProperties, EntityManagerFactoryBuilder builder) {
        return builder
                // 设置数据源
                .dataSource(secondDataSource)
                // 设置jpa配置
                .properties(jpaProperties.getProperties())
                // 设置hibernate配置
                .properties(jpaProperties.getHibernateProperties(new HibernateSettings()))
                // 设置实体包名
                .packages("com.vipshop.parrot.job.offline.model")
                // 设置持久化单元名，用于@PersistenceContext注解获取EntityManager时指定数据源
                .persistenceUnit("secondPersistenceUnit")
                .build();
    }

    /**
     * 获取实体管理对象
     *
     * @param factory 注入名为secondEntityManagerFactory的bean
     * @return 实体管理对象
     */
    @Bean(name = "secondEntityManager")
    public EntityManager entityManager(@Qualifier("secondEntityManagerFactory") EntityManagerFactory factory) {
        return factory.createEntityManager();
    }

    /**
     * 获取从库事务管理对象
     *
     * @param factory 注入名为secondEntityManagerFactory的bean
     * @return 事务管理对象
     */
    @Bean(name = "secondTransactionManager")
    public PlatformTransactionManager transactionManager(@Qualifier("secondEntityManagerFactory") EntityManagerFactory factory) {
        return new JpaTransactionManager(factory);
    }
}
```

## 7 总结和注意事项
至此，这两个数据源同时都可以使用了。指定不同的业务逻辑使用不同的数据源都是通过数据源和JPA中的package配置（本文已经在相关段落有说明）。下面是需要注意的地方：

**（a）配置数据源的时候，一定要在类名上带上@Configuration，否则会提示:**
```shell
Resource Annotation: No qualifying bean of type [javax.sql.DataSource] is defined: expected single matching bean but found 2
```
这里实际上就是spring没有将数据源当作配置加载的缘故。


**（b)在配置model的时候，一定要在类名上带上@Entity，否则会提示：**
```shell
Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: Not a managed type:
```
这里的意思就是按照JPA的规范，将modelv注册为JPA要使用到的entity。

**（c）如果不想使用JPA操作数据模型，可以直接使用下面的方法**

```java
    @PersistenceContext(unitName = "secondPersistenceUnit")
    private EntityManager entityManager;

    Query q = entityManager.createNativeQuery("select * from agent_pool");
    List<Object[]> authors = q.getResultList();
    logger.info("====>>>>" + authors.toArray().toString());

    for (Object[] o: authors) {
        for (Object j: o) {
            logger.info(j.toString());
        }

    }
```

构造entityManager来执行任意的SQL（不用定义model）。其中unitName就是用来指定想使用哪个数据源来执行SQL。这个unitName名字就定义在数据源JPA 的.persistenceUnit()属性中。 
