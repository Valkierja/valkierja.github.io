---
title: 指令集模仿设计笔记 (1) Call(E8)和Call(FF)
poc: true
categories:
  - - 硬件
tags: []
id: '537'
date: 2021-08-26 06:46:01
---

E8为near relative CALL,

FF为absolute CALL

前者Op/En基于D,操作数在机器码层面为signed Offset

后者Op/En基于M

这样就解决了CPU设计过程中两个重要问题:

1.RELATIVE CALL的方向是如何确定的；答，操作数是有符号数

2.如下代码是如何编译的；答， near relative CALL 00 00 00 00

```
CALL @F; 
@@:
mov eax,eax;
```