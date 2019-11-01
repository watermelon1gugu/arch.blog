---
title: C++ wcout的奇妙问题
date: 2019-11-02 01:11:16
tags: C++
---

## 问题阐述

![3A9948DBCAE1D0D3D09E8D70E47E7598](https://user-images.githubusercontent.com/25349066/68043446-49bdf280-fd10-11e9-9dac-f2aabcb9b45c.png)

![14E430F4F12B1283F053419A31FA83D1](https://user-images.githubusercontent.com/25349066/68043454-4c204c80-fd10-11e9-8273-07a8aa4fda64.JPG)

![72B1700E9D0D8872F6AA6A5E347AB452](https://user-images.githubusercontent.com/25349066/68043459-4dea1000-fd10-11e9-8395-0d63db6e74b1.JPG)

大概就是

```C++
wchat_t s[3] = {'a','b','c'};
wcout<<s;
```

会导致之后的其他输出都不会被输出

## 问题排查

这样的问题当然要从源码下手了

首先查看其对应的<<重载函数

![image](https://user-images.githubusercontent.com/25349066/68043750-f7310600-fd10-11e9-845d-92bd5b95361a.png)

可以看到其调用了__put_character_sequence()函数，传入了os、str和length(str)

继续深入

![image](https://user-images.githubusercontent.com/25349066/68043784-14fe6b00-fd11-11e9-900b-8e1c3793ef20.png)

貌似在函数开头是对传入参数进行param check的代码，出于直觉（偷懒），直接对if结果进行断点调试。

果不其然，原代码

```C++
if (__pad_and_output(_Ip(__os),
                                 __str,
                                 (__os.flags() & ios_base::adjustfield) == ios_base::left ?
                                     __str + __len :
                                     __str,
                                 __str + __len,
                                 __os,
                                 __os.fill()).failed())
```

会因为if判断为false而执行代码

```C++
__os.setstate(ios_base::badbit | ios_base::failbit);
```

而__pad_and_output()函数逻辑为

```C++
ostreambuf_iterator<_CharT, _Traits>
__pad_and_output(ostreambuf_iterator<_CharT, _Traits> __s,
                 const _CharT* __ob, const _CharT* __op, const _CharT* __oe,
                 ios_base& __iob, _CharT __fl)
{
    if (__s.__sbuf_ == nullptr)
        return __s;
    streamsize __sz = __oe - __ob;
    streamsize __ns = __iob.width();
    if (__ns > __sz)
        __ns -= __sz;
    else
        __ns = 0;
    streamsize __np = __op - __ob;
    if (__np > 0)
    {
        if (__s.__sbuf_->sputn(__ob, __np) != __np)
        {
            __s.__sbuf_ = nullptr;
            return __s;
        }
    }
    if (__ns > 0)
    {
        basic_string<_CharT, _Traits> __sp(__ns, __fl);
        if (__s.__sbuf_->sputn(__sp.data(), __ns) != __ns)
        {
            __s.__sbuf_ = nullptr;
            return __s;
        }
    }
    __np = __oe - __op;
    if (__np > 0)
    {
        if (__s.__sbuf_->sputn(__op, __np) != __np)
        {
            __s.__sbuf_ = nullptr;
            return __s;
        }
    }
    __iob.width(0);
    return __s;
}
```

猜测变量ob 为ouput begin 变量op为output position 变量oe为output end

调试发生，在代码段

```C++
    __np = __oe - __op;
    if (__np > 0)
    {
        if (__s.__sbuf_->sputn(__op, __np) != __np)
        {
            __s.__sbuf_ = nullptr;
            return __s;
        }
    }
```

因为\_\_np>0且\_\_s.\_\_sbuf\_->sputn(\_\_op, \_\_np) != __np

导致\_\_s.__sbuf_ = nullptr;

并返回结果.failed()

时间原因有待继续深究

个人猜测为发现输入结尾没有\0导致param check失败

同时__os.setstate(ios_base::badbit | ios_base::failbit);后，wcout将无法继续输出。

需要wcout.clean()清除错误状态

