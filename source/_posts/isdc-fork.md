---
title: isdc官网开发环境配置
date: 2018-08-20 17:29:28
tags:	
	- isdc
---

# isdc-ssm 后端

官网后端使用SpringBoot + SpringMVC +MyBatis框架进行开发

<!--*more*-->

## mysql

执行以下指令

```
sudo apt-get install mysql-server

sudo apt-get isntall mysql-client
 
sudo apt-get install libmysqlclient-dev
```

如果系统为ubuntu18.04 参考[Ubuntu18.04 Mysql无法登录问题解决方案](https://www.arch1tect.cn/2018/08/13/mysql-access-denied/)进行进一步配置

使用 mysql -u root -p 登录mysql后 执行指令 `create database isdc character set utf8;`

下载isdc.sql文件，使用 using isdc;进入到isdc数据库，使用`source isdc.sql`导入isdc.sql(注意isdc.sql文件路径填写) 

## 后端项目配置

1. 进入https://github.com/watermelon1gugu/isdc_ssm 将项目fork到自己账号上，然后使用`git 	clone https://github.com/your_github_name/isdc_ssm`将项目clone到本地。

2. 使用IDEA打开项目文件夹

   ![](https://user-images.githubusercontent.com/25349066/44333520-94ec3c80-a4a1-11e8-8fe0-31f4cd5bf678.png)

   等待IDEA自动配置Maven依赖。

3. 修改文件application.properties,将其中

   ```properties
   spring.datasource.username= ***
   spring.datasource.password= *******
   ```

   修改为本地的mysql用户名与密码

4. 在IDEA菜单栏中 view->tool window->database打开数据库连接

   ![](https://user-images.githubusercontent.com/25349066/44333817-7aff2980-a4a2-11e8-8ec1-45d123d9ca62.png)

   选择 新建->data source->mysql

   ![](https://user-images.githubusercontent.com/25349066/44333938-cfa2a480-a4a2-11e8-9f26-d32da491c1bd.png)

   database中输入isdc 并写入正确的User与password 测试连接通过后确定。

5. 后端项目采用 mybatis通用mapper + mybatis generator的形式 并使用注解配置

   配置详细参考:

   [mybatis中使用通用mapper作为DAO层](https://www.arch1tect.cn/2018/08/18/mapper/)

   [在SSM中使用mybatis-generator生成DAO层](https://www.arch1tect.cn/2018/08/16/mybatis-generator/)（ps:目前来看，使用mybatis-generator时，将pom.xml下jstl依赖注释掉，不然会出冲突）

   [MyBatis框架基于Annotation注解的一对多关联映射](https://www.arch1tect.cn/2018/08/19/mybatis-advanced-mapping/)

6. 配置完成后 启动IsdcSsmApplication即可

# isdc-ng5前端项目配置

isdc-ng5为网站主界面前端项目，使用angular开发

## 安装nodejs和npm

### 安装python-software-properties

首先需要安装依赖包python-software-properties。

```
$ sudo apt-get install python-software-properties
12
```

### 添加PPA

网站deb.nodesource.com维护了nodejs的各版本安装包的PPA，我们可以从该网站上下载执行导入。

```
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
12
```

如果提示没有安装curl，需要先安装curl。 
当前6.x版本为比较稳定的版本，我们可以根据自己的需要选择安装不同的版本。

### 安装nodejs和npm

接下来安装nodejs，安装完成之后npm也自动安装好了。

```
$ sudo apt-get install nodejs
```

### 安装 angular-cli

```
npm install -g @angular/cli
```

## 项目Clone

与isdc-ssm同理，将[isdc-manage-system](https://github.com/amaoamao/isdc-manage-system)项目fork到自己账号上并clone到本地。

## npm install

进入项目文件夹

```
cd isdc-ng5
npm install
```

## run

```
npm run start
```

> 打开浏览器输入localhost:4200即可查看网站

![](https://user-images.githubusercontent.com/25349066/44334810-2315f200-a4a5-11e8-8e00-8f9f0f04a0ef.png)

# isdc-manage-system前端项目配置

isdc-manage-system为管理员后台的前端程序，使用vue开发

## 项目Clone

与以上项目同理

clone到本地后安装npm与vue-cli

```
cd isdc-manage-system
sudo npm install
sudo npm install -g vue-cli
```

## 运行

在终端中进入项目文件夹 ，输入

```
npm run dev
```

即可运行

# 设置chrome浏览器跨域

首先到[谷歌商店上](https://chrome.google.com/webstore/search/Allow-Control-Allow-Origin?hl=zh-CNhttps://chrome.google.com/webstore/search/Allow-Control-Allow-Origin?hl=zh-CN) 下载

![image](https://user-images.githubusercontent.com/25349066/44382549-e9e28e00-a547-11e8-9822-936d4cfaeceb.png)

点击添加,我这里是已经添加了.

添加成功后启动该插件,在谷歌浏览器的右上角有一个图标,点击后出现如下界面:

![_017](https://user-images.githubusercontent.com/25349066/44382485-a25c0200-a547-11e8-9e31-130c0d31ef47.png)

开启即可跨域