---
title: Ubuntu18.04 Mysql无法登录问题解决方案
date: 2018-08-13 00:24:41
tags: 
	- ubuntu18.04
	- mysql
---

# BB

重装ubuntu18.04后，安装mysql发现安装过程中没有出现设置root密码的过程，而后无法登录mysql，网上教程众说纷纭，但尝试后都没有效果。

在尝试各种解决方案两天后终于解决了这个问题。

<!--*more*-->

# 解决方案

（1）使用 sudo 权限进入数据库

```
sudo mysql -u root
```

（2）删除原 root 用户

```
DROP USER 'root'@'localhost';
```
（3）新建 root 用户并指定密码

```
CREATE USER 'root'@'%' IDENTIFIED BY 'passwd';
```
（4）赋权

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
```

（5）更新权限

```
FLUSH PRIVILEGES;
```

然后使用 mysqll -u root -p 输入密码即可登录。