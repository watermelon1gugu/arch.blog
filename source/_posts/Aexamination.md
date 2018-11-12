---
title: 记一次惨痛的笔试
date: 2018-11-12 11:56:58
tags: 笔试	
---

# 前言

昨天做了今日头条的笔试，作为第一次参加正儿八经的笔试，有点紧张，有点糟糕，吸取教训，共勉。



# 第一题

给定一个数组 例如 （2 -1 3  4 5 -9 -2）将数组按照正负间隔输出 例如（2 -1 3 -9 4 -2 5），若有一方数字过多，将多出数字均放在末尾。

code：

```java
import java.util.Scanner;

public class Main2 {
    public static int a_size = 0;
    public static int b_size = 0;

    public static void exam(int[] a, int[] b) {
        int length = a_size + b_size;
        int aIndex = 0, bIndex = 0;
        for (int i = 0; i < length; i++) {
            if (aIndex < a_size && bIndex < b_size) {
                System.out.print(i % 2 == 0 ? a[aIndex++] : b[bIndex++]);
                System.out.print(" ");
            } else if (aIndex >= a_size) {
                for (; bIndex < b_size; bIndex++) {
                    System.out.print(b[bIndex]);
                    System.out.print(" ");
                }
                break;
            } else if (bIndex >= b_size) {
                for (; aIndex < a_size; aIndex++) {
                    System.out.print(a[aIndex]);
                    System.out.print(" ");
                }
                break;
            }
        }
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        while (in.hasNextLine()) {//注意while处理多个case
            int[] a = new int[10000000];
            int[] b = new int[10000000];
            a_size = 0;
            b_size = 0;
            String str = in.nextLine().toString();//用nextLine（）可以读取一整行，包括了空格，next（）却不能读取空格
            if (!str.isEmpty()) {
                String arr[] = str.split(" ");//拆分字符串成字符串数组
                //int c[] = new int[arr.length];
                for (String it : arr) {
                    int temp = Integer.parseInt(it);
                    if (temp < 0) {
                        b[b_size++] = temp;
                    } else {
                        a[a_size++] = temp;
                    }
                }
                exam(a, b);
            }
        }

    }
}
```

以上为后来重新写的代码，在笔试中，开考10-20分钟便写出了第一题，但提交过程中一直显示case通过率为0%。在尝试了无数IO方式后依然无果。原因猜测：在原先代码中，因为题目未告诉数据范围而使用大小可变的ArrayList作为数据容器，但是ArrayList在数据量庞大时的扩容时间是巨大的，可能造成了Time limit out.



# 第二题

一副从1到n的牌，每次从牌堆顶取一张放桌子上，再取一张放牌堆底，直到手里没牌，最后桌子上的牌是从1到n有序，设计程序，输入n，输出牌堆的顺序数组.

例如

输入：

```
1 //序列行数
1 3 5 4 2
```

输出 

```
1 2 3 4 5
```

解法（去除IO后）:

很简单的逆向操作，但笔试中仍然因为IO或者其他不明原因未得分。。。

```java
import java.util.Arrays;
import java.util.LinkedList;
import java.util.Objects;

public class Main {
    public static void main(String[] args) {
        LinkedList<Integer> list = new LinkedList<Integer>(Arrays.asList(2,4,5,3,1));
        System.out.println(exam(list));
    }

    private static LinkedList<Integer> exam(LinkedList<Integer> list){
        LinkedList<Integer> res = new LinkedList<Integer>();
        for(Integer it : list){
            res.add(0,it);
            if(!Objects.equals(it, list.getLast()))
            ex(res);
        }
        return res;
    }
    private static void ex(LinkedList<Integer> list){
        list.add(0,list.get(list.size()-1));
        list.remove(list.size()-1);

    }
}
```



实名diss某“微软员工”的错误做法 [](https://www.jianshu.com/p/fa3abe4e2531?open_source=weibo_search)

