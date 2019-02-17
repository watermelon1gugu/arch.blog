---
title: makefile学习笔记（持续更新）
date: 2019-01-07 19:09:02
tags: makefile
---



g++:

-Wall : 输出所有的警告信息

-O : 编译时进行优化

-g : 表示编译debug版本

-c : 只编程成目标文件

``` 
g++ -c file2.cpp
```

-o : 输出

``` bash
g++ file1.o file2.o -o helloworld
```

-E 预处理

```bash
g++ -E helloworld.cpp -o helloworld.i
```

-S 汇编文件

``` bash
g++ -S helloworld.i -o helloworld.s
```

-Lpath : 表示path目录中搜索库文件

-Ipath : 表示在path目录中搜索头文件

-ltest : 查找链接库

```bash
g++ -o main main.cpp -L. -lmymath
```

-fPIC : 表示编译为位置独立的代码。不使用此选项的话，编译后的代码是位置相关的，所以动态载入时通过是通过代码负责的方式来满足不同进程的需求，不能真正达到代码共享的目的。

``` bash
g++ -fPIC -c add.cpp -o add.o
```

makefile:

​	wildcard函数 : make规则中，通配符将会被自动展开 但在变量的定义和函数引用时，通配符将会失效。这种情况如果需要通配符有效 就需要使用函数wildcard

​	$(wildcard PATTERN...) 

``` bash
SOURCES = $(wildcard *.c *.cpp)
```

​	patsubst函数 : 用于匹配替换，有3个参数。第一个是需要匹配的样式，第二个表示用什么来替换，第三个是一个需要被处理的由空格分割的列表

```bash
$(patsubst %.c,%.o,$(dir) )
```

是指用patsubst把$(dir)中的变量符号后缀是.c的全部替换为.o

```bash
OBJS = $(patsubst %.c,%.o,$(patsubst %.cpp,%.o,$(SOURCES)))
```

是指用patsubst把$(SOURCES)中的所有.c .cpp变成.o 形成一个新的文件列表 存入OBJS变量中



这几句命令表示把所有的.c、.cpp文件编译成.o文件

``` bash
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
%.o: %.cpp
	$(XX) $(CFLAGS) -c $< -o $@
```

①$@ 扩展成当前规则的目的文件名

②$<扩展成依赖列表中的第一个依靠文件

③而$^扩展成整个依赖的列表(除掉了里面所有重复的文件名)