---
title: 如何从_start函数跟到main函数
poc: true
categories:
  - - CTF
  - - '%e4%ba%8c%e8%bf%9b%e5%88%b6'
    - RE
tags: []
id: '98'
date: 2021-05-21 08:09:29
---

## 背景

刚刚手把手带女朋友从start函数调试到main，然后突然发现自己没有写过这方面的教程，正好写一下

PS：在main函数之前的函数可以具体细分为多种类型的函数，本文一律统称为\_start函数

## 正文

首先要明确start函数当中都包含什么内容，然后我们才能意识到当前函数体到底是start还是main

根据笔者的经验，对于start函数的内容，我们可以首先思考一下，直接使用汇编写成的程序（只含有程序主逻辑） ，可以运行吗？当然可以；比如：

```
mov eax,3
mov ebx,4
add eax,ebx  ;a=a+b
ret ;然后返回a
```

那么我们再思考一下，既然主程序可以运行，那么\_start函数的作用是什么

根据笔者自身理解，他们的作用是为程序主逻辑创造一个更好的氛围

比如：了解当前平台和环境情况，以便在可能的情况下运行优化版的函数逻辑，或者是弹出报错“不支持当前平台”，然后退出程序

又或者是拉起一个GUI窗口以便后续资源（比如按钮类，菜单栏类）的加载

亦或者是经由操作系统内核向MMU通信，告知其“有一个新的程序需要申请堆栈”

如果是debug版程序，还可能会导入导出与调试相关的文件信息

诸如此类：1.获取当前运行环境的信息（比如运行平台等） 2.了解程序需要哪些资源（比如GUI环境等），并向操作系统申请、加载 3.获取当前程序可能不需要，但有许多程序需要的信息（比如系统时间等），以便不时之需 4.导入导出调试相关信息，或者是一些检测、处理exception的函数

主要是以上四大类

其次我们还需要明确main函数里面有什么

或者说是，CTF逆向题当中的main函数有什么

大概率来讲，会有一个输入提示“welcome to XXX CTF”或者是“please input your flag”等，然后是gets或者cin等函数的调用，然后是一个加密函数（大概率包含xor等位运算），最后是一个cmp，然后结束

所以一旦看到目标字符串（“welcome to XXX CTF”或者是“please input your flag”等）被调用（Xref）就基本可以判断当前call就是main call

### 例子

下载链接：

https://www.ksroido.art/helloworld/

首先来到EP

![](https://www.ksroido.art/wp-content/uploads/2021/05/image-1024x576.png)

接下来的内容，我们默认两个原则：1. 除非x64dbg标出注释表明当前函数内部包含了大量系统API，或是通过其他迹象表明我们还在\_start函数内部，否则我们会跟进所有call和jmp 2. 如果当前函数不是main,则运行至步出(不然对每个函数的分析都要写这一句话233)

首先遇到`call 0x0040270C`,步入,发现几个辨识度极高的\_start函数会包含的API

![](https://www.ksroido.art/wp-content/uploads/2021/05/image-1.png)

遇到`call 0x00402524`,步入,发现做了一些看不懂的操作,然后函数就退出了(没有看见调用字符串)

遇到`call 0x004024F4`,发现一个辨识度极高的API`HeapCreate`,继续

![](https://www.ksroido.art/wp-content/uploads/2021/05/image-2.png)

遇到`call 0x00402367`,发现辨识度极高的API,继续

![](https://www.ksroido.art/wp-content/uploads/2021/05/image-3.png)

遇到`call 0x00401F0B`,发现`call eax`但实际上并未执行该分支

![](https://www.ksroido.art/wp-content/uploads/2021/05/image-4.png)

遇到`call 0x00401CB7`,发现一个辨识度极高的API,继续

![](https://www.ksroido.art/wp-content/uploads/2021/05/image-5.png)

诸如此类,来到call 401000;立即警觉这一辨识度极高的地址

进入后发现messageBoxW,从而找到入口点