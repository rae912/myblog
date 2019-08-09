---
layout: post
title: "Java Spring Boot Mybatis配置多个MySQL源"
subtitle: "Java Spring Boot Mybatis With Multi MySQL Source"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
  - MySQL
---

最近工作中，需要在后端工程中连接多个MySQL。后端使用的Java的Spring Boot开发。默认使用的Mybatis做ORM。折腾了一下，费了些时间，把结果记录一下：

默认MySQL配置：
```java
import com.github.pagehelper.PageHelper;
import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import tk.mybatis.spring.mapper.MapperScannerConfigurer;

import javax.sql.DataSource;
import java.util.Properties;

import static com.vipshop.parrot.core.ProjectConstant.*;

/**
 * Mybatis & Mapper & PageHelper 配置
 */
@Configuration
public class MybatisConfigurer {

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        factory.setTypeAliasesPackage(MODEL_PACKAGE);

        //配置分页插件，详情请查阅官方文档
        PageHelper pageHelper = new PageHelper();
        Properties properties = new Properties();
        properties.setProperty("pageSizeZero", "true");//分页尺寸为0时查询所有纪录不再执行分页
        properties.setProperty("reasonable", "true");//页码<=0 查询第一页，页码>=总页数查询最后一页
        properties.setProperty("supportMethodsArguments", "true");//支持通过 Mapper 接口参数来传递分页参数
        pageHelper.setProperties(properties);

        //添加插件
        factory.setPlugins(new Interceptor[]{pageHelper});

        //添加XML目录
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        factory.setMapperLocations(resolver.getResources("classpath:mapper/*.xml"));
        return factory.getObject();
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactoryBean");
        mapperScannerConfigurer.setBasePackage(MAPPER_PACKAGE);

        //配置通用Mapper，详情请查阅官方文档
        Properties properties = new Properties();
        properties.setProperty("mappers", MAPPER_INTERFACE_REFERENCE);
        properties.setProperty("notEmpty", "false");//insert、update是否判断字符串类型!='' 即 test="str != null"表达式内是否追加 and str != ''
        properties.setProperty("IDENTITY", "MYSQL");
        mapperScannerConfigurer.setProperties(properties);

        return mapperScannerConfigurer;
    }

}
```

多个MySQL配置：
主要使用如下的配置来指明数据库连接初始化所使用的配置。配合`application.properties`文件中不同的配置前缀，Spring会自动配置并初始化
```java
@Bean(name = "parrotDataSource")
@ConfigurationProperties("spring.datasource.parrot")
```

第一个数据库链接：(这里的配置文件中的前缀为：spring.datasource.parrot，后续用到的数据连接别名：parrotDataSource)
```java
package com.vipshop.parrot.configurer;

import com.github.pagehelper.PageHelper;
import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import tk.mybatis.spring.mapper.MapperScannerConfigurer;

import javax.sql.DataSource;
import java.util.Properties;

import static com.vipshop.parrot.core.ProjectConstant.*;

/**
 * Mybatis & Mapper & PageHelper 配置
 */
@Configuration
public class MybatisConfigurer {

    @Bean(name = "parrotDataSource")
    @ConfigurationProperties("spring.datasource.parrot")
    @Primary
    public DataSource parrotDataSource(){
        return DataSourceBuilder.create().build();
    }

    @Bean
    @Primary
    public SqlSessionFactory sqlSessionFactoryBean(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        factory.setTypeAliasesPackage(MODEL_PACKAGE);

        //配置分页插件，详情请查阅官方文档
        PageHelper pageHelper = new PageHelper();
        Properties properties = new Properties();
        properties.setProperty("pageSizeZero", "true");//分页尺寸为0时查询所有纪录不再执行分页
        properties.setProperty("reasonable", "true");//页码<=0 查询第一页，页码>=总页数查询最后一页
        properties.setProperty("supportMethodsArguments", "true");//支持通过 Mapper 接口参数来传递分页参数
        pageHelper.setProperties(properties);

        //添加插件
        factory.setPlugins(new Interceptor[]{pageHelper});

        //添加XML目录
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        factory.setMapperLocations(resolver.getResources("classpath:mapper/parrot/*.xml"));
        return factory.getObject();
    }

    @Bean
    @Primary
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactoryBean");
        mapperScannerConfigurer.setBasePackage(MAPPER_PACKAGE);

        //配置通用Mapper，详情请查阅官方文档
        Properties properties = new Properties();
        properties.setProperty("mappers", MAPPER_INTERFACE_REFERENCE);
        properties.setProperty("notEmpty", "false");//insert、update是否判断字符串类型!='' 即 test="str != null"表达式内是否追加 and str != ''
        properties.setProperty("IDENTITY", "MYSQL");
        mapperScannerConfigurer.setProperties(properties);

        return mapperScannerConfigurer;
    }

}
```

第二个数据库链接(这里的配置文件中的前缀为：spring.datasource.etl，后续用到的数据连接别名：etlDataSource)
```java
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

@Configuration
// 扫描 Mapper 接口并容器管理
@MapperScan( basePackages = {EtlMybatisConfigurer.PACKAGE}, sqlSessionFactoryRef = "eltSqlSessionFactory")
public class EtlMybatisConfigurer {
    @Value("${spring.datasource.etl.driver-class-name}")
    String driverClassName;

    static final String PACKAGE = "com.vipshop.parrot.dao.etl";
    static final String MAPPER_LOCATION = "classpath:mapper/etl/*.xml";

    @Bean(name = "etlDataSource")
    @ConfigurationProperties(prefix="spring.datasource.etl")
    public DataSource etlDataSource(){
        return DataSourceBuilder.create().driverClassName(driverClassName).build();
    }

    @Bean(name = "etlTransactionManager")
    public DataSourceTransactionManager eltTransactionManager() {
        return new DataSourceTransactionManager(etlDataSource());
    }

    @Bean(name = "eltSqlSessionFactory")
    public SqlSessionFactory eltSqlSessionFactory(@Qualifier("etlDataSource") DataSource etlDataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(etlDataSource);
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(EtlMybatisConfigurer.MAPPER_LOCATION));
        return sessionFactory.getObject();
    }
}
```
这里必须注意的是，在`basePackages`中显式的知名要绑定的dao
```java
// 扫描 Mapper 接口并容器管理
@MapperScan( basePackages = {EtlMybatisConfigurer.PACKAGE}, sqlSessionFactoryRef = "eltSqlSessionFactory")
```

此时Spring Boot启动日志可能会有Warn，大意是没有给第二个数据连接指定`driver-class-name`，即显式的说明是`MySQL`，因此需要在代码中指定driverClassName。
```java
spring.datasource.etl.driver-class-name=com.mysql.jdbc.Driver
```
配置文件中明确指定MySQL。


```java
    @Value("${spring.datasource.etl.driver-class-name}")
    String driverClassName;

    //省略若干代码
    public DataSource etlDataSource(){
        return DataSourceBuilder.create().driverClassName(driverClassName).build();
    }
```
初始化代码中引用配置文件的相关配置项。


