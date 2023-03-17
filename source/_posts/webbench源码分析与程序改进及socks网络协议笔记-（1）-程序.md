---
title: webbench源码分析与程序改进及SOCKS网络协议笔记 （1） 环境搭建，程序入手，修复少量因兼容性而出现的bug
poc: true
categories:
  - - CTF
tags: []
id: '525'
date: 2021-08-25 04:34:48
---

依赖项为ctag，跟着文档安装即可

导入文件，于是便发现了很多代码风格很差的细节

首先是个人建议把socks.c的后缀改为.h，然后再include

因为在现代IDE下，.c后缀的头文件会被双重编译，而且就算是命令行编译可以通过，我也认为头文件不应该用.c后缀的文件

然后笔者这里是用VS在Windows上搭建CentOS8.2远程编译环境，需要手动下载gdb和g++

期间没遇到什么问题，略过

然后由于VS对搜索头文件深度的问题，include <signal.h>文件内定义的数据结构无法正确语法高亮

需要在原项目的基础上直接手动添加

```
#include <bits/sigaction.h>
#defineh_addrh_addr_list[0]
```

第二行的#define h\_addr h\_addr\_list\[0\] 需要特别说明一下，原定义是

```
#ifdef __USE_MISC
#defineh_addrh_addr_list[0] /* Address, for backward compatibility.*/
#endif
```

但是这个if加上去之后会影响VS的识别，这里是为了防止定义失败的一种保障的

\_\_USE\_MISC 宏对于主流Linux操作系统而言，如笔者的CentOS是无需考虑的，直接删除if即可

然后笔者这里做了函数名的一些小改动

![](https://www.ksroido.art/wp-content/uploads/2021/08/image-34.png)

Socket改为fSocket，避免和当代Linux系统函数撞名（webbench是上世纪的项目）

至此，前期操作都已完成，接下来就是添加一些现代化的功能，并改进代码行文