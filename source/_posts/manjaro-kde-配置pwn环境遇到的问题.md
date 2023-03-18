---
title: manjaro KDE 配置pwn环境遇到的问题
poc: true
categories:
  - - '二进制'
    - PWN
tags: []
id: '1192'
date: 2021-10-07 13:29:31
---

## Konsole无法唤起pwndbg/唤起新窗口后无法正确识别shebang

折腾半天，最终看了一些Konsole的源码，解决方法很简单，不用装任何额外软件

只需要在python脚本中

```
context.terminal=['konsole','--separate','-e','python']
```

即可解决所有问题

## 透明代理

https://archlinuxstudio.github.io/ArchLinuxTutorial/#/advanced/transparentProxy

跟着这个配就行

但是笔者环境下kde有个小坑

最终解决发办法是设置yakuake的脚本为

```
if (sh -c "pgrep -x qv2ray" > /dev/null)
then
    /bin/zsh
else
    cgnoproxy Applications/Qv2ray_39a851d832fd26df077eb99ae46b99f7.AppImage
fi
```

从而让脚本开机自启动且后台运行

## 开机启动小键盘

```
sudo vim /etc/sddm.conf

Numlock=none
# 改成
Numlock=on
```

## vscode无法登录

vscode有一段代码是基于gnome的逻辑

KDE因此无法登录，解决办法为

```
yay -S qtkeychain gnome-keyring
```