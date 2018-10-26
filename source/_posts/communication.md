---
title: 进程间通信-管道(pipe)
date: 2018-09-01 22:52:42
tags:
	- 进程间通信
---



#    进程间通信

每个进程各自有不同的用户地址空间,任何一个进程的全局变量在另一个进程中都看不到，所以进程之间要交换数据必须通过内核,在内核中开辟一块缓冲区,进程A把数据从用户空间拷到内核缓冲区,进程B再从内核缓冲区把数据读走,内核提供的这种机制称为进程间通信。

不同进程间的通信本质：进程之间可以看到一份公共资源；而提供这份资源的形式或者提供者不同，造成了通信方式不同，而 pipe就是提供这份公共资源的形式的一种。

<!--*more*-->

# 匿名管道

## 管道的创建

管道是由调用系统pipe()函数来创建

```c
#include <unistd.h>
int pipe (int fd[2]);
//返回:成功返回0，出错返回-1     
```

 fd参数返回两个文件描述符,fd[0]指向管道的读端,fd[1]指向管道的写端。fd[1]的输出是fd[0]的输入。

## 管道如何实现进程间的通信

（1）父进程创建管道，得到两个⽂件描述符指向管道的两端

（2）父进程fork出子进程，⼦进程也有两个⽂件描述符指向同⼀管道。

（3）父进程关闭fd[0],子进程关闭fd[1]，即⽗进程关闭管道读端,⼦进程关闭管道写端（因为管道只支持单向通信）。⽗进程可以往管道⾥写,⼦进程可以从管道⾥读,管道是⽤环形队列实现的,数据从写端流⼊从读端流出,这样就实现了进程间通信。 

![](https://user-images.githubusercontent.com/25349066/44986940-0656df80-afb8-11e8-94cc-44f21ac67a4d.png)

## 示例代码

```c
 #include<stdio.h>
 #include<unistd.h>
 #include<string.h>
 #include<sys/wait.h>
 int main()
 {
     int _fd[2];
     int ret = pipe(_fd);//创建管道
     if(0 == ret)
     {
         printf("create pipe success.\n");
     }
     else
     {
         printf("create pipe failure.\n");
         return 0;
     }

     pid_t pid = fork();//创建子进程
     if(pid == -1)
     {
         perror("fork");
         return 1;
     }
     else if(pid == 0)//child
     {
         close(_fd[0]);//子进程关闭管道的读端
         while(1)
         {
             char *str = "i am a child,i am writing.";
              ssize_t s = write(_fd[1],str,strlen(str));
              if(s == -1)
              {
                  perror("write");
                  return 2;
              }
              sleep(1);
         }
     }
     else//father
     {
         close(_fd[1]);//父进程关闭管道的写端
         while(1)
         {
             char buff[1024];
             printf("i am a father,i am reading.\n");
             ssize_t s = read(_fd[0],buff,sizeof(buff)-1);
             if(s > 0)
             {
                 buff[s] = 0;
                 printf("read success:%s\n",buff);
             }
             else if(s == -1)
             {
                 printf("read failure:");
                 perror("read");
                 return 3;
             }
         }
         wait(NULL);
     }
     return 0;
 }
```

## 管道读取数据的四种的情况

（1）读端不读，写端一直写 

![](https://user-images.githubusercontent.com/25349066/44987509-0526b200-afba-11e8-963e-b07cef6ee257.png)

（2）写端不写，但是读端一直读 

![](https://user-images.githubusercontent.com/25349066/44987512-0b1c9300-afba-11e8-9652-a81355efede8.png)

（3）读端一直读，且fd[0]保持打开，而写端写了一部分数据不写了，并且关闭fd[1]。

![](https://user-images.githubusercontent.com/25349066/44987536-18398200-afba-11e8-8ff9-bfbe306ebd16.png)

如果一个管道读端一直在读数据，而管道写端的引⽤计数⼤于0决定管道是否会堵塞，引用计数大于0，只读不写会导致管道堵塞。

（4）读端读了一部分数据，不读了且关闭fd[0]，写端一直在写且f[1]还保持打开状态。

![](https://user-images.githubusercontent.com/25349066/44987545-2091bd00-afba-11e8-8179-c0a3a3f96289.png)

## 管道特点

1. 管道只允许具有血缘关系的进程间通信，如父子进程间的通信。
2. 管道只允许单向通信。
3. 管道内部保证同步机制，从而保证访问数据的一致性。
4. 面向字节流
5. 管道随进程，进程在管道在，进程消失管道对应的端口也关闭，两个进程都消失管道也消失。

## 特殊情况

1. 如果所有指向管道写端的文件描述符都关闭了,而仍然有进程从管道的读端读数据,那么管道中剩余的数据都被读取后,再次read会返回0,就像读到文件末尾一样

2. 如果有指向管道写端的文件描述符没关闭，而持有管道写端的进程也没有向管道中写数据,这时有进程从管道读端读数据,那么管道中剩余的数据都被读取后,再次read会阻塞,直到管道中有数据可读了才读取数据并返回。

3.  如果所有指向管道读端的文件描述符都关闭了,这时有进程指向管道的写端write,那么该进程会收到信号SIGPIPE,通常会导致进程异常终止。

4.  如果有指向管道读端的文件描述符没关闭,而持有管道写端的进程也没有从管道中读数据,这时有进程向管道写端写数据,那么在管道被写满时再write会阻塞,直到管道中有空位置了才写入数据并返回。



