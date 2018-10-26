---
title: Java volatile关键词使用与原理分析
date: 2018-08-08 01:25:13
tags: volatile
---

# volatile的作用

volatile提供了一种解决有序性与可见性问题的方案.并且保证单次读/写操作的原子性,比如long和double一类64位变量类型.

<!--*more*-->

## 实现可见性

 可见性问题主要指一个线程修改了某共享变量值时,另一个线程无法立即看到.引起此类问题的主要原因是每个线程都拥有自己的一个cache即线程工作内存.使得线程可能不会在第一时间内将结果写入主存中.

 例如以下例子.

```
public class VolatileTest {
    int a = 1;
    int b = 1;

    public void change(){
        a = 3;
        b = a;
    }

    public void print(){
        if(b == 3 && a == 1){
            System.out.println("The wrong is happened");
        }
    }

    public static void main(String[] args) {
        while (true){
            final VolatileTest test = new VolatileTest();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.change();
                }
            }).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.print();
                }
            }).start();
        }
    }
}
```

 运行结果如下

![volatile可见性](https://user-images.githubusercontent.com/25349066/42735468-3363ec46-8887-11e8-9884-56322049811f.png)

 在正常情况下,应该只有 a=1,b=1与a=3,b=3两种情况,但是在运行代码时却出现了b=3,a=1的情况,其实a与b都已经完成了赋值,但是b写入了主存中,a却只保存在了cache中,没有第一时间写入主存,造成了错误的发生.

 若对a,b变量添加volatile关键字,在a,b变量发生修改时,便会在第一时间写入主存中,解决此类问题.

## 保证原子性

 在 Java® Language Specification 中,有以下描述

> 17.7 Non-Atomic Treatment of double and long For the purposes of the Java programming language memory model, a single write to a non-volatile long or double value is treated as two separate writes: one to each 32-bit half. This can result in a situation where a thread sees the first 32 bits of a 64-bit value from one write, and the second 32 bits from another write. Writes and reads of volatile long and double values are always atomic. Writes to and reads of references are always atomic, regardless of whether they are implemented as 32-bit or 64-bit values. Some implementations may find it convenient to divide a single write action on a 64-bit long or double value into two write actions on adjacent 32-bit values. For efficiency’s sake, this behavior is implementation-specific; an implementation of the Java Virtual Machine is free to perform writes to long and double values atomically or in two parts. Implementations of the Java Virtual Machine are encouraged to avoid splitting 64-bit values where possible. Programmers are encouraged to declare shared 64-bit values as volatile or synchronize their programs correctly to avoid possible complications.

 大意为出于Java编程语言存储器模型的目的，对非易失性long或double值的单次写入被视为两个单独的写入：每个32位一半写入一次。这可能导致线程从一次写入看到64位值的前32位，而从另一次写入看到第二次32位的情况。 volatile和long值的写入和读取始终是原子的。 对引用的写入和读取始终是原子的，无论它们是作为32位还是64位实现。 某些实现可能会发现将64位长或双值上的单个写操作划分为相邻32位值上的两个写操作很方便。为了效率，这种行为是特定于实现的; Java虚拟机的实现可以自由地以原子方式或分两部分执行对long和double值的写入。 鼓励实现Java虚拟机以避免在可能的情况下拆分64位值。建议程序员将共享的64位值声明为volatile或正确同步其程序以避免可能的复杂情况。

 出于效率方面的考虑,java将long与double这种64位的数据类型分为了上下32位,所以在对long与double类型数据进行操作时,出现了线程从一次写入看到64位值的前32位，而从另一次写入看到第二次32,两个32位数值并不来自同一次操作的情况.但java会保证volatile类型变量的单次读写操作是原子的,故在long与double前添加volatile关键字可以保证对于64位类型变量读写操作的原子性.

 但是volatile并不能保证复合操作的原子性,例如i++是i+1与i=i+1的复合操作,volatile并不能保证此类操作的原子执行.

## 防止重排序

在java中,实例化一个对象可以分为三个步骤

1. 为对象分配内存空间
2. 初始化对象
3. 将内存空间的地址赋给对应的引用

但是由于操作系统可以对指令进行重排序,可能导致将内存空间的地址赋给对应的引用的操作发生在初始化对象之前,导致一个未初始化对象的引用暴露在外,引发不可预料的后果.在对象前加上volatile关键字后,将不会出现此类问题

# volatile的实现

接下来解释volatile对应作用的实现

## 可见性实现

 导致线程间数据不可见问题的原因是因为线程本身并不直接与主存进行数据的交互,而是通过线程的工作内存来完成相应的操作.

 所以实现volatile变量的可见性便直接通过解决这个问题:

1. 修改volatile变量是会强制将修改后的值刷新到主存中
2. 修改volatile变量后会导致其他线程工作内存中对应的变量值失效.因此需要在使用该变量值是从主存中重新读取.

## 有序性实现

 java中对于happen-before的定义如下

> Two actions can be ordered by a happens-before relationship.If one action happens before another, then the first is visible to and ordered before the second.

 可以通过之前发生的关系来排序两个动作.如果一个动作发生在另一个动作之前，则第一个动作在第二个动作之前可见并且在第二个动作之前排序.即如果 a happen-before b,则a所做的任何操作对b来说都是可见的,并且a的操作排序都在b之前.

 同时java对于happen-before有以下规则

> • Each action in a thread happens before every subsequent action in that thread.
> • An unlock on a monitor happens before every subsequent lock on that monitor.
> • A write to a volatile field happens before every subsequent read of that volatile.
> • A call to start() on a thread happens before any actions in the started thread.
> • All actions in a thread happen before any other thread successfully returns from a join() on that thread.
> • If an action a happens before an action b, and b happens before an action c, then a happens before c.

其中规定对volatile字段的写入发生在每次后续读取该易失性之前,保证了volatile对象初始化时不会因为重排序问题造成错误的发生.