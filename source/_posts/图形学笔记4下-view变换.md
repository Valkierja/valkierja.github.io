---
title: 图形学笔记4下 view变换
poc: true
categories:
  - - 图形学笔记
tags:
  - 图形学
date: 2023-04-09 19:47:15
---

承接自上半笔记

### view transformation

也就是相机的变换

考虑现实生活中的拍照片

首先需要让被拍摄对象摆好位置(进行model transformation) 然后让摄影师找个好位置(view transformation)然后拍照(投影 projection) 合起来简称 MVP变换



<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304091953234.png" alt="image-20230409195313168" style="zoom:50%;" />



### 相机定义

相机的定义如下

![image-20230409195532703](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304091955730.png)

位置, 视方向, 向上方向(因为相机可以绕自身中轴线旋转)



### 相对运动

当相机和其他东西一起以相同方式运动时, 得到的图片不会变化

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092000326.png" alt="image-20230409200004274" style="zoom:67%;" />

也就是说, 坐标系是一个可以相对的概念, 因此为了讨论方便, 在讨论视点变换时, 我们认为:

**相机永远处于原点, 相机永远向-z轴看, 相机永远以y为向上方向**

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092004987.png" alt="image-20230409200405956" style="zoom:50%;" />

如何把相机设置为上述状态? 方法如下

![image-20230409200618485](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092006514.png)

首先位置变换到原点, 然后旋转g和t到-Z和Y

这样就对齐了具体操作请看下一小节

### 相机归位操作

设最终归位所需矩阵为 $$M$$

首先是平移变换



