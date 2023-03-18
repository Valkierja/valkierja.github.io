---
title: 大端序Windows PE文件逆向关键函数
poc: true
categories:
- [信息安全, 二进制]
tags: []
id: '615'
date: 2021-08-30 22:54:38
---

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172114249.png)

图中的函数为大小端序转换函数

遇到了一个题目是出题人自己的壳，而且壳解压过程中还把默认小端序的PE文件的一部分堆栈转换成了大端序，操作太骚了

再往后面是TEA加密的非标准实现，看WP也不懂，姑且放弃了