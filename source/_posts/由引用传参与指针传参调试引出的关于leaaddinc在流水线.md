---
title: 由引用传参与指针传参调试引出的关于lea,add,inc在流水线,分支预测上的优越性比较
poc: true
categories:
  - - '二进制'
    - RE
tags: []
id: '1268'
date: 2021-11-01 12:49:22
---

## 背景

昨天才发现按引用传参是C++的功能,C语言没有按引用,略微有点在意,顺便调试一下,发现了一些感觉值得记录的,有一点疑问的地方

## 正文

先声明一下,我之前并没有过一步步调试引用传参的经历,在我之前的淳朴想法当中,按指针传参和引用传参是一样的,对那个地址的内容解引用就可以了

不过这种淳朴的想法效率其实太低了,测试源码如下

```
#include<stdio.h>
int a = 10;
int* p=&a;
int foo(int a){
    a++;
    return a;
}
int baar(int &a){
    a++;
    return a;
}
int ham(int *a){
    (*a)++;
    return *a;
}
int main(){
    int c;
    c = foo(a);
    c = baar(a);
    c = ham(&a);
    return 0;
}
```

编译指令如下

```
 gcc -m32 -g -o refer main.cpp
```

实际情况为了把数据操作尽可能留在CPU寄存器内部

实际操作居然是用地址操作符LEA完成的

```
   0x565561d8 <baar(int&)+17>    mov    eax, dword ptr [ebp + 8]
   0x565561db <baar(int&)+20>    mov    eax, dword ptr [eax]
   0x565561dd <baar(int&)+22>    lea    edx, [eax + 1]   //这里的eax储存的内容不是地址,而是普通的数据,这里只是利用lea做加法
   0x565561e0 <baar(int&)+25>    mov    eax, dword ptr [ebp + 8]
   0x565561e3 <baar(int&)+28>    mov    dword ptr [eax], edx
   0x565561e5 <baar(int&)+30>    mov    eax, dword ptr [ebp + 8]
```

关于加法的优化,涉及流水线和分支预测的优化

文章如下

https://stackoverflow.com/questions/12163610/why-inc-and-add-1-have-different-performances