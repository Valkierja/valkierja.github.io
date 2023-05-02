---
title: webbench源码分析与程序改进及SOCKS网络协议笔记 （3）main文件解析
poc: true
categories:
  - [信息安全, 二进制]
  - [开发, 网络]
  - [笔记, 存档]
tags: [网络, 开源项目]
id: '599'
date: 2021-08-30 18:21:29
---

通读了一遍 主要逻辑是

通过分支选择来append字符串，然后发出去

fork与sleep的配合精确控制child数量，同时保证只有一层child关系

粗略看完一遍感觉封装度太高了，宏观层面看起来很简单

拓展POST功能这茬子事先放一放，先更新pwnable靶场的通关WP