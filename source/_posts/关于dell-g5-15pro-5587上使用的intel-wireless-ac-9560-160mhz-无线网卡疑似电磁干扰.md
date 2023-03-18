---
title: DELL G5 15pro 5587上使用的Intel Wireless-AC 9560 160MHz 无线网卡疑似电磁干扰的提示
poc: true
categories:
  - [杂谈, 计算机]
  - [笔记, 存档]
tags: [DELL]
id: '1262'
date: 2021-10-30 12:57:31
---

注意，标题只说了疑似，我这边只说现象，无法给出100%的证明

事情是这样的

由于自身需求原因，笔者的戴尔G5 15pro 5587购买时选择了256GB的SSD，后来不够用了

在京东官方旗舰店购买了一根SN550 512GB （WDC WDS500G2BOC- 00PXHO，详细信息如下）自带了一块马甲和硅胶导热贴

```
已配置设备 SCSI\Disk&Ven_NVMe&Prod_WDC_WDS500G2B0C-\5&35596c80&0&000000。

驱动程序名称: disk.inf
类 GUID: {4d36e967-e325-11ce-bfc1-08002be10318}
驱动程序日期: 06/21/2006
驱动程序版本: 10.0.19041.789
驱动程序提供商: Microsoft
驱动程序部分: disk_install.NT
驱动程序等级: 0xFF0007
匹配设备 ID: GenDisk
低等级驱动程序: 
设备已更新: false
父设备: PCI\VEN_15B7&DEV_5009&SUBSYS_500915B7&REV_01\4&2b082314&0&00E8

已启动设备 SCSI\Disk&Ven_NVMe&Prod_WDC_WDS500G2B0C-\5&35596c80&0&000000。

驱动程序名称: disk.inf
类 GUID: {4d36e967-e325-11ce-bfc1-08002be10318}
服务: disk
低层筛选程序: 
高层筛选程序: 
```

在安装硅胶贴和散热马甲时，出了一些意外，硅胶贴粘了一些灰尘，导致散热马甲黏贴不稳定，会翘起一些

换用SSD结束后笔者立刻发现一个现象：蓝牙和wifi会时不时自动断开，而且一旦断开必定是同时断开（具体表现为蓝牙耳机和网络同时消失）

一开始笔者只是以为更换SSD时不小心摸到了哪个电阻电容什么的，导致损坏了什么东西

后来看了一个视频，大受启发，视频粘贴如下[【官方双语】Valve到底藏了啥？- Steam Deck拆解视频观看反应#linus谈科技\_哔哩哔哩\_bilibili​www.bilibili.com/video/BV1t44y1v7Lx](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1t44y1v7Lx)

大致内容为：Steam Deck游戏机内部，wifi模块与SSD间隔很近，互相之间有干扰，官方提示务必盖好屏蔽罩

同时我注意到本机的wifi蓝牙模块物理意义上距离SSD较近

后续我尝试移除马甲，一切问题消失

再次尝试安装马甲，问题具有可重现性（已尽可能做到控制变量）