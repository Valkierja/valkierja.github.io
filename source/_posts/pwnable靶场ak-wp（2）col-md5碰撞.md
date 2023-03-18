---
title: pwnable靶场ak-WP（2）col MD5碰撞
poc: true
categories:
  - - CTF
tags: []
id: '620'
date: 2021-08-31 14:21:01
---

## 背景

这个题目之前第一次写的时候指针学的不扎实，有非常多的问题，这次打算不适用IDA的F5直接分析汇编

得到了很多经验

## 正文

连接，ls查看，发现关键文件col，下载回来本地，IDA分析

关键函数.text:08048494 Check()

关键参数重命名后如下

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181055772.png)

注意橙色箭头的ADD，正是这个操作导致IDA误报了数据类型为int，这让我在第一次做这个的时候非常麻烦，其实这里是直接对地址操作，滑套理念，把所有东西都看作是纯粹的DW数据，只是一个纯粹的数字罢了，这样思考就方便理解多了

同时注意红色箭头的后续跟进操作

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181056535.png"  />

xref找到hashcode

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181056185.png)

这里是简单的sum碰撞

几个容易混的要点，一个地址代表一个字节，一个地址是一个最小单位，一个字节是有操作意义的最小单位，32位总线每次传输一个双字（DWord，四字节），一个双字指针指向一个字节，但代表了四个字节全体

另外一个很重要的是，大小端的字节序和栈增长方向这两个东西，前者是相对于元素内部的字节顺序，后者相对于元素本身的顺序，后者把元素看作了质点，前者研究系统内部，不能看作是质点

## 补充

补一个argv的方向问题，本题当中挺重要的，这里讨论的前提是默认状态下32位Linux系统

假设有一个程序main.elf，对他argv传参 \\xc8\\xce\\xc5\\x06 即：

```
./main.elf \xc8\xce\xc5\x06
```

那么谁先入栈argv?

答案是最右侧的\\x06，同时栈向下增长，最后最低位是\\xc8

用自然语言书写顺序就是06C5CEC8h

所以可以确定，argv参数入栈和一般的函数参数入栈顺序一样，都是先push最右边的参数

## WP脚本

```

from pwn import *

a = p32(113626824)
b = p32(113626828)
payload = a*4 + b
print (payload)
p=process(['./col',payload])
print(p.recvall())
```

## 补充2

recv系类函数返回值是str，需要加一句print

C语言程序的argv参数是不支持**标准输入流命令**的，需要使用getopt