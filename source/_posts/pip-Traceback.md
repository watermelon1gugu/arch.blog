---
title: pip Traceback (most recent call last) 解决方案
date: 2018-08-18 18:30:49
tags: pip
---



# 问题描述

```c
Traceback (most recent call last):
  File "/usr/bin/pip3", line 11, in 
    sys.exit(main())
  File "/usr/lib/python3/dist-packages/pip/__init__.py", line 215, in main
    locale.setlocale(locale.LC_ALL, '')
  File "/usr/lib/python3.5/locale.py", line 594, in setlocale
    return _setlocale(category, locale)
locale.Error: unsupported locale setting
```

<!--*more*-->

# 解决方案

在 Linux 终端输入以下命令，即可

```
export LC_ALL=C
```



