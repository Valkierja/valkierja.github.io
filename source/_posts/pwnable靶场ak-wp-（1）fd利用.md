---
title: pwnable靶场ak-WP （1）FD利用
poc: true
categories:
  - - CTF
tags: []
id: '604'
date: 2021-08-30 22:36:05
---

连接上去之后ls先看一看

发现三个文件，其中一个没有权限

分析二进制文件fd

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181050197.png)

可以看出需要使用fd0

直接输入4660即可

即可获取flag

## 补充

补一个fd的笔记

之前一直很奇怪为什么read还需要指明fd

翻了一下文档就明白了

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181050137.png)

“from fd”