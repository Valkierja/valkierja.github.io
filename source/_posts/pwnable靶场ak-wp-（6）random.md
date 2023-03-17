---
title: pwnable靶场ak-WP （6）random
poc: true
categories:
  - - CTF
  - - ctf
    - 二进制
tags: []
id: '703'
date: 2021-09-04 00:48:41
---

题目原理比较简单，主要是补两个笔记

## 补充1

首先，众所周知，rand函数的种子固定时是产生的序列就是固定的

但是在不同操作系统环境下，这个序列不一定是相同的

主流Linux发行版rand的第一个值是0x6b8b4567，而Windows是41

线性同余算法不一定能达到周期上限，如果要达到周期上限，需要模除右操作数与同余式尾项互质

## 补充2

补一个pwndbg笔记，内容有限，仅放一些基础

启动时传参文件路径

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-6.png)

或者在启动后用file传参

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-7.png)

`b`下断点，加\*的数字表示地址，不加\*的数字表示函数名（但是实际上C类语言并不允许纯数字的函数名）

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-8.png)

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-9.png)

`disassemble`用来反汇编

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-10.png)

`delete`或者`d`用来删除断点

缺省删除所有断点，可以加参数，删除特定断点

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-11.png)

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-12.png)

`info b`查看当前所有断点

`r`运行程序

`reg`查看寄存器

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-13-1024x752.png)

可以看到EAX(RAX)的值是0x6b8b4567

这就是rand的固定第一个值

另外查看堆栈：

`x/nfu <addr>`或者`stack`

x 是 examine 的缩写

n表示要显示的内存单元的个数

f表示显示方式, 可取如下值  
x 按十六进制格式显示变量。  
d 按十进制格式显示变量。  
u 按十进制格式显示无符号整型。  
o 按八进制格式显示变量。  
t 按二进制格式显示变量。  
a 按十六进制格式显示变量。  
i 指令地址格式  
c 按字符格式显示变量。  
f 按浮点数格式显示变量。

u表示一个地址单元的长度  
b表示单字节，  
h表示双字节，  
w表示四字节，  
g表示八字节

另外还可以用stack来查看堆栈

然后是各种参数中含有寄存器时，寄存器都需要$符号

比如

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-15.png)

  

另外VScode远程调试是真的舒服

![](https://www.ksroido.art/wp-content/uploads/2021/09/image-14-1024x576.png)

需要改一下最大保存行数

笔者这里直接改成十万行，这下子就够用了233