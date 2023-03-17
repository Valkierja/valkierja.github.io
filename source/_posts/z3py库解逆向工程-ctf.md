---
title: Z3py库解逆向工程 CTF
poc: true
categories:
  - - CTF
  - - '%e4%ba%8c%e8%bf%9b%e5%88%b6'
    - RE
tags: []
id: '331'
date: 2021-08-17 02:03:51
---

## 正文

先安装，建议用whl手动安装而非apt、pip，同时注意给管理员权限，以免初始化失败

然后有几个z3的坑点，一个是bitvector和bitvectors这类变量，一个是单数一个是复数

如果一次性初始化很多变量就用带s的

同时注意只有bitvector才可以进行位运算

初始化变量时，建议把传参字符串和外部变量名统一，如果不统一有可能可以解释器通过，但是一定会对编辑器的语法高亮造成一定困扰

另外，需要先check才能打印结果，在命令行中结果可以直接输出，但是在编辑器界面需要加一句print

## 示例

https://www.ksroido.art/wp-content/uploads/2021/08/千层饼.zip

首先是通过简单分析发现一系列很深的函数

通过查看Num1和Num2的xref发现只有四个函数对他们进行了特殊操作

这个想法是这样的：

首先理解ecx是this指针，然后观察xref里面哪个寄存器出现了很多次，就首先排除与那个寄存器相关的操作，因为这个操作大概率是 duplicated，然后注意观察cmp调用

最终选出四个约束函数

添加到z3里面

```
from z3 import *
x,y=BitVecs('x y',32)
s=Solver()
s.add(y/2-x == -107702)
s.add(y & 1 == 0)
s.add(y ^ 333509 == x)
s.add(881778^666>=y)
print(s.check())
print(s.model())
```

顺便说一下from xxx import \*

和

import xxx的区别

前者相当于include -r

后者相当于include --none

从学习py开始困扰我的问题终于解决了。。。