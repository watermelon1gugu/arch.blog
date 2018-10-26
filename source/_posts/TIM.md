---
title: 在ubuntu18.04中使用TIM
date: 2018-08-20 02:36:47
tags:	
	- TIM
---

# Github地址

https://github.com/wszqkzqk/deepin-wine-ubuntu 个人感觉为目前最为成功的linux版TIM，尚未发现任何问题。

<!--*more*-->

# 安装

1. 在控制台输入 `git clone https://github.com/wszqkzqk/deepin-wine-ubuntu.git` 将项目克隆到本地

2. 进入deepin-wine-ubuntu文件夹 运行`./install.sh` 
3. 进入http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.office/ 下载TIM最新deb包
4. 进入deb包所在文件夹 运行 `dpkg -i deepin.com.qq.office_2.0.0deepin4_i386.deb `(deb包文件名以实际下载文件名为准)

# 运行

在软件列表中选择tim打开即可运行

![TIM](https://user-images.githubusercontent.com/25349066/44312166-5e251080-a426-11e8-8beb-c02bf8c1294b.png)



