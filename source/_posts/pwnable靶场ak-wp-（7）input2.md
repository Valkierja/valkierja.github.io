---
title: pwnable靶场ak-WP （7）input2
poc: true
categories:
  - - CTF
  - - ctf
    - 二进制
tags: []
id: '770'
date: 2021-09-06 22:16:12
---

先补一个pwntool传参的笔记和一个python构造list的好方法

## 补充1

pwntool的argv传参有个很神奇的地方

他甚至能篡改argv\[0\]的值,没翻源码,实现暂时未知,不过不影响

创建方法为

`connect.process(list : argv,str : path)`

## 补充2

如果有一个很长的list,其中大部分元素的值为垃圾填充,只有小部分为有用值,那么错误做法是append,太傻了

正确做法是生成一个全为填充值的list,然后抓几个特殊的地方出来

## WP

the very vital point that I didn't notice that need be metioned at first

If you excu an ELF then you will carry identify of the ELF's owner

the main\_check func need a file named"\\x0a",so we need create such file,while we don't have permission in the.floder

so we need goto tmp floder and use utility `ln` to create soft-link,which only need link dest-floder writing permission

```
ln -fs  ~/flag flag
ln -fs  ~/input input
```

for now on,every step operates on tmp-link ver file

and hold on a min ,le't talk about permission of create a hard-link and soft-link file

if we'd like to create a hard-link ,the sufficient condition is,we have read+write permission on target file,if `fs.protected_hardlinks == true` and have write & **execu** permission of destdir.if\``fs.protected_hardlinks == 0` no permission needed on target file,even 000 is okay.

what surprise me is execu permission,its purpose is preventing a situation that's similar to this quzz--if hack have access to tmp floder,which normally is gloabally permitted,the hack can execu or open wrong file,due to relative path

for soft-link the requires is lower,only need source-dir have execu and dest-dir have write+execu

for target file ,its permission is no need to care

come back to main topic

after create link we need notice that relative path have a relative vulnerability ,make soft-link to cheat execu with wrong file

secondly,there is a sock programming needed ,considering webbench project ,this time we try use C to pwn rather than pwntool

## poc

（under construction..）