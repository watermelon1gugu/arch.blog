---
title: do{}while(0)的使用技巧
date: 2019-02-17 14:24:39
tags:
---

# do{}while(0)的使用技巧

## 避免宏定义错误

在定义较为复杂的宏定义时，比如

```c++
#define DOSOMETHING() foo1();foo2();
```

由于代码将会被展开（被当做两行）

所以在面对

```C++
if(something)
    DOSOMETHING();
```

这种情况的时候会出现无论判断条件是什么，foo2()都会执行的问题。

如果使用

```c++
#define DOSOMETHING() {foo1();foo2();}
```

则展开后的代码为

```c++
if(something){
    foo1();
    foo2();
};
```

大括号后面跟着一个分号，在老式的编译器中会出现语法错误的情况。

所以在没有do/while(0)的情况下，几乎无法保证我们的多语句宏总能正确执行

使用do/while(0)来定义如下

```c++
#define DOSOMETHING() \
    do{ \
        foo1();\
        foo2();\
    }while(0)\
```

do保证了大括号中逻辑能够被正确执行。while(0)保证了代码只被执行一次。

## 避免使用goto控制程序流

在一些时候中，我们可能想要在某些条件下跳过之后的语句执行。最直接的方式是使用goto

```c++
int foo(){
    dosomething...;
    if(error)
        goto END;
    dosomething...;
    if(error)
        goto END;
    dosomething...;
END:
    return 0;
}
```

由于goto不符合软件工程的结构化，而且有可能使得代码难懂，所以很多人都不倡导使用，这个时候我们可以使用do{...}while(0)来做同样的事情：

```c++
int foo(){
    do{
        dosomething...;
        if(error)
            break;
        dosomething...;
        if(error)
            break;
        dosomething...;
    }while(0);
    return 0;
}
```

这里将函数主体部分使用do{...}while(0)包含起来，使用break来代替goto，后续的清理工作在while之后，现在既能达到同样的效果，而且代码的可读性、可维护性都要比上面的goto代码好的多了。

