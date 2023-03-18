---
title: Linux 进程权限划分
poc: true
categories:
  - - CTF
  - - CTF
    - [信息安全, 二进制]
tags: []
id: '774'
date: 2021-09-06 17:59:19
---

很奇怪pwnable的题目为什么可以用system cat flag 但是flag文件明明没有权限

查了一圈各种离奇回答都有,看来在需要的时候不太容易找到标准答案,这次一并找到,补个笔记

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181101133.png)

错误文案示例

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181101079.png)

描述不清,容易理解错误

其实是real ID和effective ID的关系

如果是执行一个权限为X的可执行文件,一般情况下会把进程权限视作可执行文件所有者,而非执行可执行文件的用户

当然,如果要实现后者情况也很简单

另外还有一种情况,需要介于root和user之间的权限,可执行权限会标记为s

这时候会被允许操作root才能操作的一些文件,但是携带的权限并非root权限,而是用户本身的权限