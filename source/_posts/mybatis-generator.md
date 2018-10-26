---
title: 在SSM中使用mybatis-generator生成DAO层
date: 2018-08-16 01:11:32
tags: mybatis-generator
---

## 配置generatorConfigxml

在maven工程中的resource中创建generatorConfig.xml

![](https://user-images.githubusercontent.com/25349066/44162451-f05da980-a0f2-11e8-9248-7c88c97bff52.png)

文件内容如下

<!--*more*-->

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 数据库驱动包位置 -->
    <!-- 由于在pom.xml中加入插件时已经配置数据库驱动包，所以此处不必配置了-->
    <!-- <classPathEntry location="D:\generator\mysql-connector-java-5.1.34.jar" /> -->
    <!--<classPathEntry location="E:\Database\Oracle\jdbc\lib\ojdbc14.jar" />-->
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!-- 数据库链接URL、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/****" userId="***" password="***">
        </jdbcConnection>
       <!-- <jdbcConnection driverClass="oracle.jdbc.driver.OracleDriver" connectionURL="jdbc:oracle:thin:@localhost:orcl" userId="scott" password="tiger">
        </jdbcConnection>-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!-- 生成模型的包名和位置 -->
        <javaModelGenerator targetPackage="com.ScuSoftware.Factorio.model" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- 生成的映射文件包名和位置 -->
        <sqlMapGenerator targetPackage="com.ScuSoftware.Factorio.mapping" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- 生成DAO的包名和位置 -->
        <javaClientGenerator type="ANNOTATEDMAPPER" targetPackage="com.ScuSoftware.Factorio.dao" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 要生成那些表(更改tableName和domainObjectName就可以) -->
    
        <table tableName="tableName" domainObjectName="domainObjectName" enableCountByExample="true" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true" />

      
    </context>
</generatorConfiguration>
```

## 配置maven##

在pom.xml中添加

```xml
<!-- mvn mybatis-generator:generate -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
                <configuration>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                    <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                </configuration>
                <dependencies>
                    <!-- 数据库驱动 -->
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.46</version>
                    </dependency>
                </dependencies>
            </plugin>
```

## 在Intellij IDEA添加“Run运行”选项##

### Step1：选择配置edit configuration###

![](https://user-images.githubusercontent.com/25349066/44163453-10429c80-a0f6-11e8-8db7-d6e8b1fdb61a.png)

### Step2:创建maven运行项###

![](https://user-images.githubusercontent.com/25349066/44164294-3701d280-a0f8-11e8-9eec-0aa0ad39b0a2.png)

### Step3:配置命令 mybatis-generator:generate -e

![](https://user-images.githubusercontent.com/25349066/44163592-6e6f7f80-a0f6-11e8-8582-784568b4c724.png)

### Step4:运行###

在run下拉菜单中选中mybatis-generator点击运行

![1534356471992](https://user-images.githubusercontent.com/25349066/44189865-da81d000-a156-11e8-940e-60ec7302d611.png)



