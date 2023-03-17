---
title: indirectly CALL with PLT and GOT
poc: true
categories:
  - - '%e4%ba%8c%e8%bf%9b%e5%88%b6'
    - RE
  - - ctf
    - 二进制
tags: []
id: '1256'
date: 2021-10-28 18:20:57
---

## bkgnd

FOA,English is not my first language and this is just a note for myself,if you take this post by accident please forgive my mistakes

## Main

​​

![](https://www.ksroido.art/wp-content/uploads/2021/10/image-20211028173306045-1024x278.png)

Full procedure picture is uploaded

when statically disassemble ,got\[1\] and \[2\] is 0x0000 0000.they will be filled by ldd correctly

BTW,we must notice that got and got.plt is aproximately equal to DATA section,while .plt is aproximately equal to TEXT section

resolve happen iff first call happens

resolve find global func addr by its name stored in .dynstr