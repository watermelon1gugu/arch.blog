---
title: my-hexo-dir-source-posts-hexo-YAMLException-cannot-read-a-block-mapping-entry-错误的解决办法
date: 2018-08-07 23:41:43
tags: 解决办法
---

# my hexo dir/source/_posts/hexo-YAMLException-cannot-read-a-block-mapping-entry-错误的解决办法

没想到在发出第一篇博文时就出现了问题<!--*more*-->,在运行了hexo g 后,显示了

> ERROR Process failed: _posts/博客真好玩.md
>
> YAMLException: can not read a block mapping entry; a multiline key may not be an implicit key at line 4, column 1:

在删除了md文档内全部内容后却能成功生成,后来发现



```
title: 博客真好玩
date: 2018-07-14 09:25:18
tags:博客
```

中tags:后未加空格,添加上后成功解决.每一个配置项都是名称+冒号+空格＋设置参数配置而成,故在冒号后必须加上空格.