---
title: D触发器的使能端和清零端
poc: true
categories:
  - [笔记,课程,计组]
tags: [数电,硬件,课程笔记]
id: '1101'
date: 2021-09-17 16:08:26
---

## 同步清零和异步清零

这里的同步异步是相对于CLK而言的

同步的"同"是与CLK相同有效边沿信号频率

异步则是一旦有行为信号输入,则立刻产生对应行为,不等待下一次CLK

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-70.png)

## 使能端

对于触发器而言，使能信号在整体逻辑上，相当于和CLK相与；但是实际上不会直接用与门处理，因为这样会减速，CLK信号可能会异步

与门可以理解为放行或者封锁

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-67.png)

上述是好理解，但是错误的做法（尽量不要在CP信号线上增加任何逻辑运算元器件）

下述是正确做法，利用**选择器**，筛选信号，并把一路作为回馈信号

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172057722.png)

但只需要简单画为En引脚就行

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-69.png)