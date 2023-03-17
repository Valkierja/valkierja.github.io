---
title: Linux网络程序逆向小笔记 (2)
poc: true
categories:
  - - CTF
  - - ctf
    - 二进制
tags: []
id: '821'
date: 2021-09-08 22:26:28
---

有的时候，一些相似的网络数据包结构体类型IDA会分析错误，在本例中，sockaddr\_in被错误分析为 sockaddr ，看得很不方便

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-29.png)

修改方法还挺简单的（但是有点难以想象是如何保证的，毕竟是字符串匹配）

在最上面的声明，找到对应的变量

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-24.png)

然后按Y

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-25-1024x132.png)

直接改名字

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-26-1024x132.png)

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-27.png)

然后就分析正确了

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-28.png)