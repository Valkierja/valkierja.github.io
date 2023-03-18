---
title: x64dbg 保存更改到可执行文件
poc: true
categories:
  - [信息安全, 二进制]
  - [笔记,短笔记]
tags: []
id: '64'
date: 2021-04-21 20:45:55
---

背景介绍接不说了,直接正文讲解遇到的问题和解决方案

本文示例程序来自《逆向工程核心原理》的hello world 实验

找到MessageBoxW函数之后

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172107192.png)

找到目标字符串`4092A0:L"Hello World!"`

Ctrl+G跳转4092A0发现数据被识别成代码

解决方案为在下方内存窗口当中查找,但是由于内存窗口默认显示外部dll的调用堆栈情况

因此需要在内存布局选项卡当中进入.rdata段,然后跳转至4092A0

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172111941.png)

选中需要修改的部分（注意小端模式）然后ctrl+E（edit）

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172110139.png)

修改完成后再ctrl+P（patch）

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172110253.png)

选择修补文件，另存为后缀名exe文件即可