---
title: 句柄详解笔记 (3) Windows系统API GetModuleHandle 逆向笔记
poc: true
categories:
  - - CTF
tags: []
id: '446'
date: 2021-08-24 01:28:26
---

示例代码如下(C++),有一些无效的内容,比如string什么的,无视即可

```
#include<iostream>
#include<windows.h>
#include<winternl.h>
#include<string>
#include<libloaderapi.h>
#include<winuser.h>
using namespace std;

int main() {
HANDLE a = GetModuleHandleA(NULL);
cout << a << endl;
return 0;
}
```

这里需要关闭随机基址

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172101111.png)

编译出来之后x86dbg打开,用API断点插件(https://www.52pojie.cn/thread-1384349-1-1.html)直接断在GetModuleHandle

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172101122.png)

跟跳

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172101521.png)

第一句是mov edi,edi，查了一下知识点比较多，之后单独开一篇

push保存现场

获取参数1(参数1是module名,这里是NULL表示exe程序自身)

sub扩展局部空间

test重置flag

je跟跳

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172101252.png)

出乎笔者意料之外，核心内容只有两行

查fs表30偏移

ring0下偏移30存放的指向是PEB的指针

peb+8偏移是ImageBaseAddress，也就是基址

返回eax

至于什么是PEB我觉得可以单开一篇文章，因为网上能找到的讲PEB的文章实在是太少了

能查到的文章基本全是一模一样的互相抄，找了半天最初来源居然是夜影爷爷写的23333，有点戏剧性