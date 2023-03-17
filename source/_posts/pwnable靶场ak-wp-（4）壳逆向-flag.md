---
title: pwnable靶场ak-WP （4）壳逆向 flag
poc: true
categories:
  - - CTF
  - - ctf
    - 二进制
tags: []
id: '656'
date: 2021-09-01 22:30:02
---

## 思路

直接IDA打开，发现乱码，回来查壳，查到UPX

尝试直接脱壳，无阻碍，翻一下逻辑找到明文flag