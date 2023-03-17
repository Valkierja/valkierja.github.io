---
title: and esp, 0FFFFFFF0h的作用
poc: true
categories:
  - - '%e4%ba%8c%e8%bf%9b%e5%88%b6'
    - RE
tags: []
id: '1198'
date: 2021-10-08 11:40:13
---

主要是为了抬地址，对齐

不过更重要的细节是，IDA有时会因此对esp分析错误

基于esp，而非ebp的函数体需要多加注意，最好还是去动调一下

![](https://www.ksroido.art/wp-content/uploads/2021/10/image.png)