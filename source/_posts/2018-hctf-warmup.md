---
title: 2018 HCTF Warmup
poc: true
categories:
  - [信息安全, Web]
tags: []
id: '121'
date: 2021-05-21 20:47:32
---

打开题目出现滑稽图片

F12看到提示source.php

简单分析source.php后看见hint.php

进入hint.php

提示flag在ffffllllaaaagggg文件当中

分析各个白名单过滤函数

（strpos() 助记:stringPosition）

关键点在于如何get传入含有两个问号的字符串

构造payload

`?file=source.php?file=/../../../../../../../ffffllllaaaagggg`

## 2021.5.22更新

`?file=source.php?file=/../../../../../../../ffffllllaaaagggg`

这个payload原先是直接抄别的WP的

后来误导笔者走向了一个致命的误区

下面直接说结论,跳过笔者搜索过程和做lab过程

当php服务部署在Linux操作系统上的时候，作为一门弱耦合的语言，会有如下现象：

include函数查询目录时，首先会以`/`符号为分界,当某个目录查找失败时,会将查找对象直接变为下一个`/`符号后所指定的目录,并尝试进行查找,例如:

`include (sdfsfgstd/dir1/dir2/dwgegebsdf/flag)`

其中,只有dir1和dir2是合法目录,而flag是最终flag文件,include会抛出两个warning然后打开/dir1/dir2/flag

具体到本题,payload也可以是:

`?file=source.php?sdfwegreawfw/../../../../sdvsdfsa/../ffffllllaaaagggg`