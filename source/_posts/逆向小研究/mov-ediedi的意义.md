---
title: mov edi,edi的意义
poc: true
categories:
  - [信息安全, 二进制, 逆向]
  - [开发,Windows]
  - [笔记, 存档]
tags: [Windows API]
id: '456'
date: 2021-08-24 02:14:20
---

所有WindowsAPI第一句都是mov edi,edi

有点好奇,查了一下原因,写篇笔记

本质上来讲，mov edi,edi相当于在PE中占两字节长度的NOP，消耗的CPU时钟也和NOP相同（在某些常见架构下）

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172057596.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181056031.png)

即使不相同也相近，基本不影响整体效率

提到效率，可能有人已经反应过来了

这个确实是一个在正常状态下完全没意义的指令，只会消耗CPU时钟，他唯一的意义是帮助开发者热更新和打补丁

这基本解决了我对如何实现汇编级热更新和汇编级补丁的所有疑问

参考文章如下

https://devblogs.microsoft.com/oldnewthing/20110921-00/?p=9583

选取几个要点翻译一下

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172057788.png)

我们在调用系统api的时候，并不是一进去就jmp的，而是先mov edi

这样的话，如果需要热更新这个api就只需要先在全局层面修改dll文件，宏观表现就是hook掉

至于如何修改，这里涉及到机器码长度

long jmp和mov edi,edi的机器码长度是相同的，这样就不用在热更新时修改PE头了，实际上热更新时也不可能修改PE头的节区地址，这会影响到整个文件

Windows系统DLL都有这一行，不过主流编译环境基本都可以让你自己的dll库加上这个指令，只需要在设置里面开启即可

文章还解释了一下为什么不直接用hook来热更新，说不能确定的任一调用者执行点什么的，没看懂，个人感觉从技术上应该可以做到，不过笔者也没做过hook，先按下不表，姑且认为是这样的，估计做过hook实验就明白了