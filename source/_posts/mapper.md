---
title: mybatis中使用通用mapper作为DAO层
date: 2018-08-18 02:33:48
tags: 
	- mapper
---

# 官方文档

https://github.com/abel533/Mapper/wiki

<!--*more*-->

# 环境配置

在pom.xml中添加

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.0.3</version>
</dependency>
```

# 简单示例

数据库有如下表：

```
CREATE TABLE `country` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `countryname` varchar(255) DEFAULT NULL COMMENT '名称',
  `countrycode` varchar(255) DEFAULT NULL COMMENT '代码',
  PRIMARY KEY (`Id`)
) ENGINE=InnoDB AUTO_INCREMENT=10011 DEFAULT CHARSET=utf8 COMMENT='国家信息';
```

对应的 Java 实体类型如下：

```
public class Country {
    @Id
    private Integer id;
    private String  countryname;
    private String  countrycode;

    //省略 getter 和 setter
}
```

最简单的情况下，只需要一个 `@Id` 标记字段为主键即可。数据库中的字段名和实体类的字段名是完全相同的，这中情况下实体和表可以直接映射。

**提醒：如果实体类中没有一个标记 @Id 的字段，当你使用带有 ByPrimaryKey 的方法时，所有的字段会作为联合主键来使用，也就会出现类似 where id = ? and countryname = ? and countrycode = ? 的情况。**

> 第四章会介绍代码生成器，可以自动生成上面的实体和下面的接口代码

**通用 Mapper 提供了大量的通用接口，这里以最常用的 Mapper 接口为例**

该实体类对应的数据库操作接口如下：

```
import tk.mybatis.mapper.common.Mapper;

public interface CountryMapper extends Mapper<Country> {
}
```

> 只要配置 MyBatis 时能注册或者扫描到该接口，该接口提供的方法就都可以使用。

该接口默认继承的方法如下：

- *selectOne*
- *select*
- *selectAll*
- *selectCount*
- *selectByPrimaryKey*
- *方法太多，省略其他...*

从 MyBatis 中获取该接口后就可以直接使用：

```
//从 MyBatis 或者 Spring 中获取 countryMapper，然后调用 selectAll 方法
List<Country> countries = countryMapper.selectAll();
//根据主键查询
Country country = countryMapper.selectByPrimaryKey(1);
//或者使用对象传参，适用于1个字段或者多个字段联合主键使用
Country query = new Country();
query.setId(1);
country = countryMapper.selectByPrimaryKey(query);
```

**如果想要增加自己写的方法，可以直接在 CountryMapper 中增加。**

**1. 使用纯接口注解方式时**

```
import org.apache.ibatis.annotations.Select;
import tk.mybatis.mapper.common.Mapper;

public interface CountryMapper extends Mapper<Country> {
    @Select("select * from country where countryname = #{countryname}")
    Country selectByCountryName(String countryname);
}
```

> 这里只是举了个简单的例子，可以是很复杂的查询。

# 与Mybatis-Generator相结合

## 添加maven依赖

Maven中<build>标签内添加

```xml
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
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.46</version>
                    </dependency>
                    <dependency>
                        <groupId>org.freemarker</groupId>
                        <artifactId>freemarker</artifactId>
                        <version>2.3.23</version>
                    </dependency>
                    <!--MyBatis Generator及工具-->
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.3.7</version>
                    </dependency>
                    <dependency>
                        <groupId>tk.mybatis</groupId>
                        <artifactId>mapper</artifactId>
                        <version>4.0.2</version>
                    </dependency>
                </dependencies>
            </plugin>
```

如果之前已经配置好mybatis，则适当修改<plugin>内的内容

## 配置generatorConfig.xml

在resources文件夹里创建generatorConfig.xml文件,编辑该文件进行配置，示例如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 数据库驱动包位置 -->
    <!-- 由于在pom.xml中加入插件时已经配置数据库驱动包，所以此处不必配置了-->
    <!-- <classPathEntry location="D:\generator\mysql-connector-java-5.1.34.jar" /> -->
    <!--<classPathEntry location="E:\Database\Oracle\jdbc\lib\ojdbc14.jar" />-->
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <property name="javaFileEncoding" value="UTF-8"/>
        <!--配置是否使用通用 Mapper 自带的注释扩展，默认 true-->
        <!--<property name="useMapperCommentGenerator" value="false"/>-->

        <!--通用 Mapper 插件，可以生成带注解的实体类-->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper,tk.mybatis.mapper.hsqldb.HsqldbMapper"/>
            <property name="caseSensitive" value="true"/>
            <property name="forceAnnotation" value="true"/>
            <property name="beginningDelimiter" value="`"/>
            <property name="endingDelimiter" value="`"/>
        </plugin>

        <!--通用代码生成器插件-->
        <plugin type="tk.mybatis.mapper.generator.TemplateFilePlugin">
            <property name="targetProject" value="src/main/java"/>
            <property name="targetPackage" value="isdc.isdcssm.dao"/>
            <property name="templatePath" value="generator/mapper.ftl"/>
            <property name="mapperSuffix" value="Dao"/>
            <property name="fileName" value="${tableClass.shortClassName}${mapperSuffix}.java"/>
        </plugin>

        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/isdc?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8"
                        userId="root"
                        password="xxxxxxx">
        </jdbcConnection>

        <!--MyBatis 生成器只需要生成 Model-->
        <javaModelGenerator targetPackage="isdc.isdcssm.model" targetProject="src/main/java"/>

        <table tableName="user" >
            <generatedKey column="id" sqlStatement="JDBC"/>
        </table>
      
    </context>
</generatorConfiguration>
```

与原版Mybatis Genetator有些许区别，详细参照官方文档

## 运行

运行流程参照 [在SSM中使用mybatis-generator生成DAO层](https://www.arch1tect.cn/2018/08/16/mybatis-generator/)

