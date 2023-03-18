---
title: 以消息为导向的有限状态机理念Windows程序设计（汇编） (2)三种常见消息处理机制概念及设计
poc: true
categories:
  - [信息安全, 二进制]
  - [开发, Windows]
tags: []
id: '416'
date: 2021-08-20 07:12:27
---

上一篇讲了最常见的一种：

while true循环GetMessage期间检查WM\_QUIT，并分派、处理消息

讲讲其他两种以及三种的区别（主要是性能上的）

## 正文

首先是，为什么需要GetMessage？我们既然已经引入了委托回调机制，那么完全可以让操作系统直接把消息发送给回调函数（假定回调函数已经注册了，这个不是技术难点），当回调函数发现窗口关闭时执行break，而剩下的整个主程序都是while true循环，如下所示

```
invoke CreateWindow,…
invoke ShowWindow,…
invoke UpdateWindow,…
.while dwQuitFlag == 0 ;要退出时在窗口过程中设置dwQuitFlag
.endw
invoke ExitProcess,…
```

这个问题的答案是，GetMessage是系统函数，可以让程序心甘情愿地在空闲时间将时间片移交给操作系统，将这个等待循环（while true）移交给操作系统内部，而操作系统内部的while true会更有利于整机性能，这也变相解释了为什么在空闲的时候系统中断这个进程倾向于占用接近100%的CPU

简单来说就是空闲资源要尽可能停留在操作系统层面，而非用户程序层面

所以这个形式的消息处理模块是负优化

另一种形式则是正优化（某种意义上），使用peekMessage函数，该函数与GetMessage的区别在于，GetMessage会在消息队列空闲时内陷于操作系统内核，而在相同情况下 peekMessage 会立刻返回一个NULL表示消息队列空闲，这种情况下如果采用上一篇文章的常规做法，当前时间片就会完全陷入在不处理任何工作的等待循环（在GetMessage内部），虽说 空闲资源要尽可能停留在操作系统层面，而非用户程序层面

但若空闲资源能够由用户程序利用，而非单纯停留在操作系统内部，有时候会更好，至于为什么是有时候后面会讲

如果监测到消息队列空闲，我们可以做一些诸如检查程序更新，发送错误报告之类的事情

但是不能做太大的工作，设函数foo是一个极为庞大的单线程非异步函数

```
.while TRUE
    invoke PeekMessage,addr @stMsg,NULL,0,0,PM_REMOVE
    .if eax
        .break .if @stMsg.message == WM_QUIT
        invoke TranslateMessage,addr @stMsg
        invoke DispatchMessage,addr @stMsg
        .else
        call foo  ;foo函数
        .endif
```

假想一种情况：

某一瞬间检测到消息队列空闲, 进入else空闲任务列表,执行非异步函数foo,由于该函数过于庞大,未等执行完操作系统便结束了当前时间片，更甚在foo执行过程中出现了新消息，而foo非异步，用户程序控制权内陷于其中，直到该函数返回，导致期间返回前窗口迟滞

所以这种消息队列处理逻辑设计只能说是 ”有时候会更好“