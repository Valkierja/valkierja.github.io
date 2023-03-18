---
title: 为什么hyper-V和VM conflict
poc: true
categories:
  - [笔记, 存档]
tags: [虚拟化, 环境配置]
id: '1401'
date: 2022-03-30 19:08:25
---

如果开了hyper-v那么整台Windows物理机都是hyper-V虚拟化的,hyper-v的效率和一台普通物理机的效率是差不多的

而对于VM来说就相当于运行在虚拟机里面

所以报错,不能双重虚拟化

比较麻烦的解决方法是

> 如果要用VMware，暂时关闭Hyper-v或沙盒功能，命令如下：  
> （1）以管理员身份（win + x）运行命令提示符；  
> （2）执行命令：bcdedit /set hypervisorlaunchtype off  
> （3）重启系统，运行vm即可。  
>   
>   
> 如果要恢复使用沙盒功能：  
> （1）以管理员身份（win + x）运行命令提示符；  
> （2）执行命令：bcdedit / set hypervisorlaunchtype auto  
> （3）重启系统，运行Sandbox即可。  
>   
>   
> 作者：WHOANIl  
> 链接：https://www.jianshu.com/p/73ccf4af4a67  
> 来源：简书  
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

一般沙盒就用来测试不安全软件的

所以一般保持关闭即可