---
title: 关于priority encoder芯片的简单小分析
poc: true
categories:
  - - 硬件
  - - '%e7%a1%ac%e4%bb%b6'
    - 芯片
tags: []
id: '1394'
date: 2022-02-11 04:02:36
---

## 背景

CIS221的实验

## 正文

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172110571.png)

电路图如上

首先看第一位bit0

bit0先用与/与非门控制, 当其他更高级的信号为0时与/与非门才会放过去

u13到u15的或门是选择

分别只有一个可能被选择通过

并且对应情况下会让bit1和bit2 bit3也改变

比如

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172110485.png)

u13通过去了一个信号到bit0,但是并不知道这个bit0的信号是来自谁

所以需要这个信号连到bit1等处,来进一步判断来自谁

现在问题来了....

这个该怎么从0设计呢...

毕竟我是看着视频写的

总感觉逆向很好考虑

但是正向完全想不出来