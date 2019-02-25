---
title: ubuntu使用小技巧(持续更新)
date: 2018-08-21 22:56:18
tags: ubuntu
---

# 图片批量格式转换

下边的命令将会一次性的讲当前文件夹下的所有bmp格式的图片修改为jpg格式

```
for i in *.bmp;do convert ${i} ${i%bmp}jpg;done
rm -rf *.bmp
```

<!--*more*-->

# 一键安装依赖

```
apt --fix-broken install
```

# 触摸板右键失灵问题

```
 gsettings set org.gnome.desktop.peripherals.touchpad click-method areas
```

# 安装jdk8

```
sudo add-apt-repository ppa:webupd8team/java

sudo apt-get update

sudo apt-get install oracle-java8-installer
```

# 添加sudo用户组

```bash
sudo useradd -m hadoop -s /bin/bash //创建hadoop用户
sudo passwd hadoop //设置密码
sudo adduser hadoop sudo //添加管理员权限
```

