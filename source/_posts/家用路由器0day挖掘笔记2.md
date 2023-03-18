---
title: 家用路由器0day挖掘笔记（2）HTTP协议POST报文与环境配置
poc: true
categories:
  - [信息安全, 0day]
  - [信息安全, 二进制, 固件]
tags: [0day, 固件, 路由器, 网络, 流量分析]
id: '883'
date: 2021-09-10 02:06:12
---

## 衔接

由于最终攻击目标的payload是基于POST报文的，所以我们需要对HTTP的某些部分（主要是POST）了如指掌，顺便笔者也补补课，之前学web时只是泛泛了解HTTP协议而已，希望这次的梳理能帮到大家

## 配置环境

请自行搜索并安装如下环境：

VM虚拟机，Ubuntu12.04 （配置buildroot交叉编译MIPS环境） ，Ubuntu16.04（配置buildroot交叉编译MIPS环境），IDA(及MIPS插件)，binwalk（安装图像模块，capstone模块，Ihasa模块，sasquatch模块，squashFS模块），以及QEMU-MIPS

着重说一下QEMU，这个程序是我们用来虚拟化路由器环境的虚拟机

尽可能安装高版本的QEMU（最好是latest），否则会有一些配置上的麻烦，具体情况本文不展开，可以确定的是，latest已经修复了相关bug，但是早期版本有一些小bug

### QEMU安装

这里是kali/Debian/Ubuntu下的安装

```
sudo apt install -y bc zlib1g-dev  libglib2.0-0 libglib2.0-dev libtool libsdl1.2-dev libpixman-1-dev autoconf qemu qemu-user qemu-user-static qemu-system
```

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172113244.png)

在干净的kali2021.2环境下,运行上述安装代码后，笔者没有遇到任何坑点

### 交叉编译环境

```
wget http://buildroot.uclibc.org/downloads/snapshots/buildroot-snapshot.tar.bz2
tar -jxvf buildroot-snapshot.tar.bz2
cd buildroot
sudo apt-get install libncurses-dev patch bc
make clean
make menuconfig
```

选择`Target Option`\->`Target Architecture`\->`MIPS`

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172113460.png)

这里需要选择大小端

虽然MIPS默认大端，但是我们的这个漏洞的平台采用的是小端

然后貌似就会自动修改`Target Architecture Variant`如果没有自动修改，则手动修改为Generic MIPS（通用MIPS）

然后修改`tool chain`当中的Linux内核版本，修改为本机内核版本

查看内核可以使用

`uname -r`

设置成功后退出并保存

然后

`sudo apt install -y texinfo bison flex`

然后

```
sudo make
```

等待编译结束

在干净的kali2021.2下,上述代码执行后没有遇到任何问题

此时我们cd到~/buildroot/output/host/usr/bin

可以找到mipsel-linux-gcc(把他搞个别名软连接放到系统环境里面,或者添加一个系统环境变量)

这里的el是little-endian的反写

这里我们测试一下

```
#include<stdio.h>

int main(){
    printf("hello world");
    return 0;
}



mipsel-linux-gcc -o hello ./main.c -static
```

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172112750.png)

顺便测试一下刚刚安装的qemu

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172112765.png)

行为正常

这里需要注意的一点是

qemu-mips用于运行程序,qemu-system-mips用于运行整个mips架构的操作系统

### chroot

这里需要注意一点，我们上面使用了`-static`选项，那么，如果我们要使用动态链接库该怎么做？

其实很简单

首先复制一份qemu的副本到某个地方

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172112714.png)

这样是为了环境干净和安全

然后用chroot更改目录，然后报错了(cannot find /bin/bash)

搜索了一番，加上一些笔者摸索的方法

首先我们需要把当前的shell复制到目录下，同时利用ldd工具，得到当前shell调用的lib库，吧lib库也复制过来

然后再把~/buildroot/output/host/mipsel-buildroot-linux-uclibc/sysroot下面的lib文件夹里面的东西递归-R复制过来，这就可以了

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172112282.png)

这里其实最终原理很简单，但是笔者第一次做的时候没有转过弯来：

qemu只是虚拟机，就好比VMware，他不包含任何操作系统的系统文件，而交叉编译环境包括了所有目标环境的动态库文件所以我们复制过来就行

此时作者也发现了一些很重要的问题，zsh有点不兼容问题，需要改配置文件，建议还是用Ubuntu 不要用kali

### 配置网络环境

这里借鉴了一下MrK师傅的网络配置教程，笔者之前从来也没有从0纯命令行配置TUN/TAP协议虚拟端口转发NAT连接的经验，之后会发一篇专门的教程，本系列的第三篇会稍微鸽一下

下面的代码直接复制并执行即可，在干净的Ubuntu12 Ubuntu16 ，Ubuntu21下均无问题

如果遇到了IP冲突，请手动修改，但是应该本地没有那么多监听吧

```
sudo apt-get install bridge-utils uml-utilities
sudo brctl addbr virbr0
sudo ifconfig virbr0 192.168.122.1/24 up
sudo tunctl -t tap0
sudo ifconfig tap0 192.168.122.11/24 up
sudo brctl addif virbr0 tap0
sudo dnsmasq --strict-order --except-interface=lo --interface=virbr0 --listen-address=192.168.122.1 --bind-interfaces --dhcp-range=192.168.122.2,192.168.122.254 --conf-file="" --pid-file=/var/run/qemu-dhcp-virbr0.pid --dhcp-leasefile=/var/run/qemu-dhcp-virbr0.leases --dhcp-no-override
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -p /etc/sysctl.conf
sudo iptables -t nat -A POSTROUTING -s "192.168.122.0/255.255.255.0" ! -d "192.168.122.0/255.255.255.0" -j MASQUERADE
sudo iptables -N vm-service
sudo iptables -A vm-service -j ACCEPT
sudo iptables -A FORWARD -s 192.168.122.0/24 -j   vm-service
```

第一行是安装必要环境

第二行创建虚拟端口

第三行为端口指定IP

第四行创建虚拟tap

第五行为tap指定IP

第六行连接tap和虚拟端口

第七行是配置DHCP

第八行第九行，端口转发

第十行到最后，配置NAT和防火墙

然后就可以用qemu-system-mipsel测试

```
sudo qemu-system-mipsel -M malta -kernel <kernelPATH> -hda <imgPATH> -append "root=/dev/sda1 console=tty0" -netdev tap,id=tapnet,ifname=tap0,script=no -device rtl8139,netdev=tapnet -nographic
```

测试log如下

```
root@ks:~# sudo qemu-system-mipsel -M malta -kernel vmlinux-3.2.0-4-4kc-malta -hda debian_wheezy_mipsel_standard.qcow2 -append "root=/dev/sda1 console=tty0" -netdev tap,id=tapnet,ifname=tap0,script=no -device rtl8139,netdev=tapnet -nographic
[    0.000000] Initializing cgroup subsys cpuset
[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Linux version 3.2.0-4-4kc-malta (debian-kernel@lists.debian.org) (gcc version 4.6.3 (Debian 4.6.3-14) ) #1 Debian 3.2.51-1
[    0.000000] bootconsole [early0] enabled
[    0.000000] CPU revision is: 00019300 (MIPS 24Kc)
[    0.000000] FPU revision is: 00739300
[    0.000000] Determined physical RAM map:
[    0.000000]  memory: 00001000 @ 00000000 (reserved)
[    0.000000]  memory: 000ef000 @ 00001000 (ROM data)
[    0.000000]  memory: 00675000 @ 000f0000 (reserved)
[    0.000000]  memory: 0789b000 @ 00765000 (usable)
[    0.000000] Wasting 60576 bytes for tracking 1893 unused pages
[    0.000000] Initrd not found or empty - disabling initrd
[    0.000000] Zone PFN ranges:
[    0.000000]   DMA      0x00000000 -> 0x00001000
[    0.000000]   Normal   0x00001000 -> 0x00008000
[    0.000000] Movable zone start PFN for each node
[    0.000000] early_node_map[1] active PFN ranges
[    0.000000]     0: 0x00000000 -> 0x00008000
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 32512
[    0.000000] Kernel command line: root=/dev/sda1 console=tty0
[    0.000000] PID hash table entries: 512 (order: -1, 2048 bytes)
[    0.000000] Dentry cache hash table entries: 16384 (order: 4, 65536 bytes)
[    0.000000] Inode-cache hash table entries: 8192 (order: 3, 32768 bytes)
[    0.000000] Primary instruction cache 2kB, VIPT, 2-way, linesize 16 bytes.
[    0.000000] Primary data cache 2kB, 2-way, VIPT, no aliases, linesize 16 bytes
[    0.000000] Writing ErrCtl register=00000000
[    0.000000] Readback ErrCtl register=00000000
[    0.000000] Memory: 122332k/123500k available (4596k kernel code, 1168k reserved, 1278k data, 220k init, 0k highmem)
[    0.000000] NR_IRQS:256
[    0.000000] CPU frequency 200.00 MHz
[    0.000000] Console: colour dummy device 80x25
[    0.000000] console [tty0] enabled, bootconsole disabled

Debian GNU/Linux 7 debian-mipsel ttyS0

debian-mipsel login: root
Password: 
Linux debian-mipsel 3.2.0-4-4kc-malta #1 Debian 3.2.51-1 mips

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@debian-mipsel:~# ping www.baidu.com
PING www.a.shifen.com (39.156.66.18) 56(84) bytes of data.
64 bytes from 39.156.66.18: icmp_req=66 ttl=51 time=15.9 ms
64 bytes from 39.156.66.18: icmp_req=67 ttl=51 time=18.6 ms
64 bytes from 39.156.66.18: icmp_req=68 ttl=51 time=15.7 ms
64 bytes from 39.156.66.18: icmp_req=69 ttl=51 time=16.2 ms
64 bytes from 39.156.66.18: icmp_req=70 ttl=51 time=16.0 ms
```

成功

## 修复环境

这里是本文的难点。 QEMU只能模拟MIPS-CPU环境，但是无法模拟路由器当中的**其他硬件**，这可能会导致固件导入虚拟机后无法运行，因此我们需要**重写并Hook部分so动态库**

### Hook nvRAM板载颗粒主控驱动 (/dev/nvram)

(施工中)