---
title: 使用ModelMapper实现对象到对象的映射
date: 2018-08-18 00:39:20
tags: 
	- modelMapper
---

# 为什么需要映射

应用程序通常由相似但不同的对象模型组成，其中两个模型中的数据可能相似，但模型的结构和关注点不同。对象映射可以轻松地将一个模型转换为另一个模型，从而允许单独的模型保持隔离。

<!--*more*-->

# 为什么使用ModelMapper

ModelMapper的目标是通过基于约定自动确定一个对象模型如何映射到另一个对象映射，就像人类一样 - 同时提供一个简单的，重构安全的API来处理特定用例，从而使对象映射变得容易。

## 智能

ModelMapper分析您的对象模型，以[智能地](http://modelmapper.org/user-manual/how-it-works/)确定应如何映射数据。不需要手动映射。ModelMapper为您完成大部分工作，自动投影和展平复杂模型。

## 重构安全

ModelMapper提供了一个简单，流畅的[映射API，](http://modelmapper.org/user-manual/property-mapping/)用于处理特殊用例。API是类型安全且重构安全的，使用实际代码而不是字符串引用来映射属性和值。

## 基于公约

ModelMapper使用[约定](http://modelmapper.org/user-manual/configuration/)来确定属性和值如何相互映射。用户可以创建自定义约定，也可以使用提供的约定之一。

## 扩展

ModelMapper支持与任何类型的数据模型[集成](http://modelmapper.org/user-manual/integrations)。从JavaBeans和JSON树到数据库记录，ModelMapper为您提供了繁重的工作。



# 配置#

在Maven中添加依赖

```xml
<dependency>
  <groupId>org.modelmapper</groupId>
  <artifactId>modelmapper</artifactId>
  <version>2.1.1</version>
</dependency>
```

如果是使用springBoot 则添加依赖为

```xml
<dependency>
    <groupId>com.github.jmnarloch</groupId>
    <artifactId>modelmapper-spring-boot-starter</artifactId>
    <version>1.1.0</version>
</dependency>
```



# 使用

官方文档 http://modelmapper.org/getting-started/ 比我详细多啦，就不多讲了



基本使用

```
modelMapper.map(user, UserResponse.class);
//将UserResponse映射为user
```

未完待续