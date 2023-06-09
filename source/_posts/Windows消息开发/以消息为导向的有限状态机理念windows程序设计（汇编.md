---
title: 用汇编编写消息系统 (1) 一些基本知识点、向操作系统注册一个窗口
poc: true
categories:
  - [开发, Windows]
tags: []
id: '374'
date: 2021-08-20 01:33:41
---

## 背景

会开发才会逆向，然而笔者越来越发现其实自己对实际生产环境中的Windows程序设计理念一无所知，谨以此系列文章记录

## 正文

笔者有过用最蠢的方式开发Windows窗体程序的经验，示例程序是计算器，要求是支持分数

前期觉得挺简单的，越到收尾越发现一系列问题，简单来说有以下：

如果要支持括号运算的话，手写符号顺序树太难了，于是笔者做法是托管JSengine，使用JS的eval函数

由于需要支持分数线，所以必须符号和数字分开，然后就会造成delete键的逻辑需要非常多的if，因为需要判断接下来删除的是什么，是普通符号还是分数线还是数字？

而其实Windows窗体程序并不是这样的设计逻辑，题主在开发初期找了网上一些程序源码借鉴，找到的三四份源码都是这种错误的设计逻辑，因此笔者也跟着这个逻辑写，陷入了误区

其实这里最重要的一点就是简单CUI程序和Windows窗体程序之间的一个不同点：

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181052750.png)

回到计算器的例子，用户点击任何按钮都是顺序，次数不定的，这就导致了我前期写函数逻辑，写封装很爽，一写一个爽，但是后期却需要一堆if嵌套判断调用逻辑顺序，同时如果需要加入异常输入检测的话，if数量，层数直接爆炸，完全没法继续写，最终导致重写整个逻辑

回到正确做法，正确做法应该是用一个while true把消息获取，消息处理逻辑括起来，再加一句break（当程序退出时触发）

```
.while TRUE
        invoke GetMessage,addr @stMsg,NULL,0,0
        .break .if eax == 0
        invoke TranslateMessage,addr @stMsg
        invoke DispatchMessage,addr @stMsg
.endw
```