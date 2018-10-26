---
title: 构建SpringBoot+SpringMvc+Mybatis项目
date: 2018-08-16 17:22:23
tags:	
	- SpringBoot
	- SpringMVC
	- Mybatis
---

# 生成SpringBoot项目

进入[start.spring.io](start.spring.io)中构建项目

![](https://user-images.githubusercontent.com/25349066/44201570-25faa500-a17c-11e8-951f-aeada712be99.png)

选择需要的依赖后点击构建,会生成项目文件并下载。

<!--*more*-->

# 环境配置

下载压缩包后解压并使用IDE打开文件，等待maven配置完成

## 编辑application.properties

打开src.main.resources中的application.properties文件，写入配置

```properties
#修改为8888端口，不配置默认8080端口
server.port=8888
# 数据库访问配置
# 主数据源，默认的
#mybatis.mapper-locations=classpath:mapper/mapper/*Mapper.xml
#mybatis.config-location=classpath:mapper/sqlMapConfig.xml
mybatis.type-aliases-package=isdc.model
spring.datasource.url=jdbc:mysql://localhost:3306/factorio?useSSL=false&useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# 下面为连接池的补充设置，应用到上面所有数据源中
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
spring.datasource.filters=stat,wall,log4j
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
# 合并多个DruidDataSource的监控数据
spring.datasource.useGlobalDataSourceStat=true

spring.mvc.view.prefix=/WEB-INF/view/
spring.mvc.view.suffix=.jsp
#spring.resources.static-locations=classpath:/resources/,classpath:/static/
```

## 配置Mybatis-generator

[详细流程](https://www.arch1tect.cn/2018/08/16/mybatis-generator/)

## 添加mysql配置

在pom中添加 

```
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>
```

完成以上内容，SSM项目就初步创建完成了