---
title: PE loader如何判断PE文件确实是一个PE文件？
poc: true
categories:
  - [笔记, 存档]
tags: []
id: '424'
date: 2021-08-20 07:36:30
---

只会进行四步判定就默许为PE文件，如果这种默许出现了其他错误，会在遇到非法指令时掷错退出并认为这是一个编译等过程出现一定错误的PE文件（还是PE文件）

第一步，扩展名是否为exe

第二步，文件头是否为MZ

第三步，003Ch处（e\_lfanew）的数据是否为一个合法虚拟地址

第四步，e\_lfanew的地址指向的内容是否为IMAGE\_NT\_SIGNATURE

上述过程在PE loader中一部分源码：

```
assume esi:ptr IMAGE_DOS_HEADER
.if [esi].e_magic != IMAGE_DOS_SIGNATURE
jmp _ErrFormat
.endif
add esi,[esi].e_lfanew
assume esi:ptr IMAGE_NT_HEADERS
.if [esi].Signature != IMAGE_NT_SIGNATURE
jmp _ErrFormat
.endif
invoke _ProcessPeFile,@lpMemory,esi,@dwFileSize
```