---
title: Windows PyCharm远程解释Linux PWNtools配置过程
poc: true
categories:
  - - CTF
tags: []
id: '635'
date: 2021-09-01 17:42:32
---

## 背景

在虚拟机Linux写脚本实在是太蛋疼了，虚拟机外面Windows写的话，又没有pwntool语法高亮，物理机Linux虽然笔者自己用着挺舒服，但是时不时就会需要用一下Windows独占应用，比如腾讯课堂这种沙雕玩意，对wine优化也不够好

最后配置了一下远程python解释器，pwntool挂在服务器上，有些ssh和nc操作直接走服务器专线带宽，更容易ping通

## 正文

查文章的时候看到前人的文章里面有很多报错，意外的是笔者自己没遇到任何问题

直接在解释器里面点击添加解释器-添加SSH

然后一直下一步就好了

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181053542.png)

## 补充

pwntool的ssh等连接命令的输出会有一个转啊转的ASCII动画

pycharm不支持显示那玩意

会直接卡死

笔者现在最优解是VScode配置remote develop，用下来没啥问题，很舒服

就是在切换VPN的时候会抽风一下，GitHub上面找到有人反馈了一下，但是微软没搭理emmm

总的来说小问题