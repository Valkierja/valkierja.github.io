---
title: Linux网络程序逆向小笔记 (1)
poc: true
categories:
  - - CTF
  - - CTF
    - [信息安全, 二进制]
tags: []
id: '813'
date: 2021-09-08 22:09:48
---

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181100968.png)

socket函数建立连接时，会通过地址族和传输方式自动推算出使用协议，因此第三个参数一般直接写NULL即可

然后是修改纯数字参数

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181100173.png)

类似这种

只需要修改成枚举就行了

这里出现了一个“奇怪”的东西，枚举

顺便列一下为什么使用枚举

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181100580.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181101669.png)

另外，考虑到大部分宏定义的前缀相同，实际上可以写一个插件来确定这些枚举（IDA已经有这样的接口了）

但是笔者没找到相关的插件

或许可以写一个，但是最近在搞网络固件逆向，没空学IDA插件开发，后续如果做了这个项目再补上笔记