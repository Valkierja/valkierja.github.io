---
title: Arch Manjaro KDE 无法创建输入输出后端。未知的协议“tg”
poc: true
categories:
  - - CTF
    - [信息安全, 二进制]
  - - 杂谈
tags: []
id: '1330'
date: 2021-12-09 22:29:22
---

无法打开telegram链接

解决方法：

1.确认你平时登录的用户，如果是root用户，这个帖子对你无效

2.如果平时是用普通用户的话，vim ~/.local/share/applications/   然后进Telegram配置文件

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172056064.png)

manjaro默认Vim有插件,双击就可以进去，如果你没有插件那就找找自行路径

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181101309.png)

进去后是这样的

在\[Desktop Entry\]里面任意位置写

```
MimeType=x-scheme-handler/tg;
```

即可