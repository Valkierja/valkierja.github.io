---
title: pwndbg的笔记汇总(更新中)
poc: true
categories:
  - [笔记,存档]
tags: [pwndbg, 技巧]
id: '1180'
date: 2021-10-05 16:58:04
---

## 设置flag寄存器

在32位程序下，`regs`会直接显示flag寄存器的内容，我们只需要使用

```
p/x  <num>  以十六进制打印
p/t  <num>  以二进制打印
```

进行进制转换

然后使用

```
set $eflags=   //注意eflags是复数形式
```

赋值即可

但是64位程序`regs`不显示标志寄存器,需要使用info reg

后续步骤相同

## x指令常见使用方法

```
x/<nums><font><data_width> <addr>
```

<font>：

1.  x 按十六进制格式显示变量。
2.  d 按十进制格式显示变量。
3.  t 按二进制格式显示变量。
4.  i 指令地址格式
5.  c 按字符格式显示变量。
6.  f 按浮点数格式显示变量。

<data\_width>：

1.  b表示单字节，
2.  h表示双字节，
3.  w表示四字节，
4.  g表示八字节