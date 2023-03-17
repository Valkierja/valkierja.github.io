---
title: why ebp is empty while gcc compiled with -ggdb  arg(without optimization)
poc: true
categories:
  - - '%e4%ba%8c%e8%bf%9b%e5%88%b6'
    - PWN
    - RE
tags: []
id: '1195'
date: 2021-10-08 11:37:46
---

The answer basically is "why IT'S called calling-convention rather than requirement"

for \_cdecl ,it suggest fn set-up its env and tear-up by itself

so a fn is actually can run without save ebp,since it's a special fn

while there is still a safty reason that to prevant from shellcode 'leave' get control of stack frame of \_libc\_start\_main fn