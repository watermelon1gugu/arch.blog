---
title: ubuntu18.04安装opencv 3.4.1
date: 2018-08-29 13:58:16
tags: opencv
---

# 安装相关依赖

```
 sudo apt-get install build-essential
 sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
 sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev # 处理图像所需的包
 sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev liblapacke-dev
 sudo apt-get install qt5-default qtcreator # 安装QT库
 sudo apt-get install libxvidcore-dev libx264-dev # 处理视频所需的包
 sudo apt-get install libatlas-base-dev gfortran # 优化opencv功能
 sudo apt-get install ffmpeg
```

<!--*more*-->

# 下载opencv源码

```
git clone https://github.com/opencv/opencv.git 
cd opencv 
```

# 编译opencv

```
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=Release -D WITH_QT=ON -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j8
make install
```

