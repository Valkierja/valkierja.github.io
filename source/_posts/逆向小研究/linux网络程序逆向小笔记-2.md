---
title: Linux网络程序逆向小笔记 (2)
poc: true
categories:
  - [信息安全, 二进制]

tags: [网络]
id: '821'
date: 2021-09-08 22:26:28
---

有的时候，一些相似的网络数据包结构体类型IDA会分析错误，在本例中，sockaddr\_in被错误分析为 sockaddr ，看得很不方便

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181053803.png)

修改方法还挺简单的（但是有点难以想象是如何保证的，毕竟是字符串匹配）

在最上面的声明，找到对应的变量

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181053629.png)

然后按Y

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181053729.png)

直接改名字

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054704.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054897.png)

然后就分析正确了

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054779.png)