---
title: 开发小问题及其解决方法
date: 2018-09-03 23:14:53
tags:
---



1. [spring  Date类型格式化](https://blog.csdn.net/beauxie/article/details/78552919) 

2. [springboot 2.0 配置 spring.jackson.date-format 不生效](https://blog.csdn.net/qq_30912043/article/details/80967352)

3. mysql新建utf8格式数据库

   ```sql
   CREATE DATABASE isdc DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
   
   
   CREATE DATABASE 的语法：
   CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name
   [create_specification [, create_specification] ...]
   create_specification:
   [DEFAULT] CHARACTER SET charset_name
   | [DEFAULT] COLLATE collation_name
   
   更改数据库的字符编码
   ALTER DATABASE db_name DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci
   ```
