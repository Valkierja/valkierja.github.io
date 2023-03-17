---
title: 微电路组合电路元件笔记
poc: true
categories:
  - [笔记,课程,计组]

tags: []
id: '994'
date: 2021-09-13 15:08:40
---

## 选择器

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116232.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116564.png)

## 译码器

相当于排列组合,取某一位的真值

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172115180.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116008.png)

## SR锁存器

R,S极如果输入互斥或信号，则为设置储存内容

如果设置成功后,输入0,0两极信号，则内部存储时序电路信号

如果输入1,1两极信号，则UB

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-66.png)

## D锁存器

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-42.png)

SR锁存器前面加两个与门，E是enable端，D是data

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-43-1024x185.png)

当E为1，D（输入）和Q（输出）之间就好似透明的

## D触发器

CP信号(Clock Pulse)跳变瞬间变化，C1为时钟端（数字1），是clock的cl的异写

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-44-1024x350.png)

术语称为，有效沿触发

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-45.png)

有效沿到来前称为现态，之后称为次态

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-46-1024x619.png)

两个D锁存器并联后与CLK串联

## 锁存器和触发器的区别

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-47-1024x570.png)

## 三态门

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-52.png)

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-53.png)

可以在一条总线上放多个元件而不会信号互扰