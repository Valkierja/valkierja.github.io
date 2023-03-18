---
title: 按值传递与引用传递（Function pass by value vs. pass by reference译文）
poc: true
categories:
  - [信息安全, 二进制]
  - [开发]
  - [杂谈]
tags: [cpp]
id: '15'
date: 2021-03-24 20:56:10
---

# 背景

之前写过一篇简单区分了一下指针变量和普通变量的区别

后来发现自己还是太年轻了,在外网上面看见了这篇[文章(2)](https://courses.washington.edu/css342/zander/css332/passby.html) 
感觉写的非常透彻,翻译一下作为笔记

**(未经许可,此翻译禁止以任何形式转载,摘录,二次创作)**

# 阅读本文前置知识

*   了解栈帧与简单的汇编,编译原理有助于理解本文
*   如果不了解的话建议看上面的文章(1)就行了

# 正文

我们将传入函数的参数称为实参，将接受实参并被实参赋值的参数称为形参，他们统称为参数

当我们向函数传递参数时，不同的调用方式导致的不同结果往往会令人困惑，与其称之为“规定”，不如去确切的了解一下不同调用方式在底层当中的具体细节。

回想一下，当你调用一个函数时，一块栈将被划分为帧，这篇文章讨论的关键是此函数的帧当中保存形参和局部变量的情况

根据定义，按值传递意味着函数将会在栈帧中开辟一段内存空间，用于储存实参的**值**的副本（译注：也就是所谓的**保存现场**），当你需要在函数当中使用这个参数时，你只会用到他的值的副本，而非使用其“原件”。当你只是需要**暂时使用**参数，而非**在全局范围内更改参数的值**时，请使用按值传递。

而至于引用传递，则会在栈帧中开辟一段内存空间，用于储存实参的**地址**的副本，函数内任何对形参的操作会直接作用于实参本身（译注：地址虽然是副本，但是相同的地址指向相同的内存，**实际上是同一个东西**；而对于值的副本，不同的副本占据不同的内存空间，互不影响，**实际上是不相干的东西**），当需要**在全局范围内更改参数的值**传入的参数本身时，请使用引用传递

（后面的例子并非原文例子，而是由译者给出 由于笔者主要接触的是汇编语言,于是舍弃了原文当中的C++例子,采用此汇编例子,看不懂该例子的话可以去看一下原文的例子）

按值传递：

```
mov         dword ptr [i],1h  ；待使用的两个变量
mov         dword ptr [j],2h 


push        ebp  ;生成栈帧
 mov         ebp,esp  
 sub         esp,0CCh  ;开辟内存空间
 push        ebx  ;保存现场
 push        esi  
 push        edi  
 lea         edi,[ebp-0CCh]  ;安全性填充
 mov         ecx,33h  ;安全性填充
 mov         eax,0CCCCCCCCh  ;安全性填充
 rep stos    dword ptr es:[edi]  ;安全性填充
 call swapThemByVal  ;调用函数


；swapThemByVal
 mov         eax,dword ptr [num1]  ;复制值的副本到eax
 mov         dword ptr [temp],eax 

 mov         eax,dword ptr [num2]  
 mov         dword ptr [num1],eax  

 mov         eax,dword ptr [temp]  
 mov         dword ptr [num2],eax ;由于编译优化,并未存入新开辟的内存空间当中,而是直接存入eax

mov eax ,0; 返回值
 pop         edi  ;恢复现场
 pop         esi  
 pop         ebx  
 add         esp,0CCh  
 cmp         ebp,esp  
 call        __RTC_CheckEsp   ;检查是否堆栈溢出
 mov         esp,ebp  ;销毁变量,释放子栈帧,为返回父栈帧做准备
 pop         ebp  ;恢复父栈帧
 ret  ; 返回
```

引用传递：

```
lea         eax,[j]  ;复制地址的副本
push        eax  ;将副本入栈
lea         ecx,[i]  ;同理
push        ecx  
push        ebp ;创建栈帧 
 mov         ebp,esp  
 sub         esp,0CCh  ;填充int3
 push        ebx  
 push        esi  
 push        edi  
 lea         edi,[ebp-0CCh]  
 mov         ecx,33h  
 mov         eax,0CCCCCCCCh  
 rep stos    dword ptr es:[edi]  
call        swapThemByRef 

;swapThemByRef 

mov         eax,dword ptr [num1]  ;复制地址的副本到eax
 mov         ecx,dword ptr [eax]  ;eax当中存储的是地址,间接引用这个地址
 mov         dword ptr [temp],ecx  

 mov         eax,dword ptr [num1]  
 mov         ecx,dword ptr [num2]  
 mov         edx,dword ptr [ecx]  
 mov         dword ptr [eax],edx  

 mov         eax,dword ptr [num2]  
 mov         ecx,dword ptr [temp]  
 mov         dword ptr [eax],ecx  

 mov         eax,0  ;返回值
 pop         edi  
 pop         esi  
 pop         ebx  
 add         esp,0CCh  
 cmp         ebp,esp  
 call        __RTC_CheckEsp (0BA129Eh)  
 mov         esp,ebp  
 pop         ebp  
 ret  
```

