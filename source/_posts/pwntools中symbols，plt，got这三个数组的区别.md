---
title: pwntools中symbols，plt，got这三个数组的区别
poc: true
categories:
  - [信息安全, 二进制]
  - [笔记, 存档]
tags: [技巧, pwntools]
id: '1227'
date: 2021-10-17 19:59:07
---

其实主要是回答symbols数组和plt，got数组的区别

对于pwntools而言，外部符号在symbols数组的内容一般与plt相同

对于内部符号则仅有symbols会标注，plt不负责记录

而got就是got了，也就是plt的跳转对象