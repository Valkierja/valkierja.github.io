---
title: pwn1_sctf_2016 WP
poc: true
categories:
  - - '%e4%ba%8c%e8%bf%9b%e5%88%b6'
    - PWN
tags: []
id: '1370'
date: 2022-01-11 15:43:59
---

## 背景

IDA分析错了+一些暂时悬而未决的疑点,故作记录

## 正文

核心利用点在replace

这里我不赘述了,毕竟是老题,而且是简单题,wp思路可以看看其他师傅

这里主要是讲两点

一个是,貌似文件里面的所有STL函数的第一个参数都是这个

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181050064.png)

然后进去函数后函数会自动覆盖掉arg\[0\],再把所有参数下移4个偏移

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181050278.png)

再退出函数时,会把arg\[0\]再次设为这个值

这里这两点让我很在意

但是没有找到合适的说法,编译原理学的比较菜

暂且悬置