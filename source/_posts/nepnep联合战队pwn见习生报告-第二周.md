---
title: Nepnep联合战队PWN见习生报告-第二周
poc: true
categories:
  - - '二进制'
    - PWN
tags: []
id: '1291'
date: 2021-12-03 21:39:22
---

# 正文

**（[Attack Lab](https://link.zhihu.com/?target=http%3A//csapp.cs.cmu.edu/im/labs/attacklab.tar)\[Updated 1/11/16\]）**

看题目描述我这可来劲了啊!

写ROP什么的最畅快了(大雾)

先下载题目文件[康康](https://www.zhihu.com/search?q=%E5%BA%B7%E5%BA%B7&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A438535451%7D)

有两个主体文件,总共包含五道题目

直接IDA

# 第一个文件ctarget

## 第一题touch1

先是学到了一个好东西（优秀的开发实现）

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172115050.png)

is\_checker被放在bss段，恒0

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172115282.png)

不过也不是重点，同时我很难想象啥情况下这个值为true

然后main首先来到这里

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172115435.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054507.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054586.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054202.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054141.png)

所以这里是最简单的ret2，不过！且慢！

先来checksec检查一下

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116124.png)

这里有canary

但是蹊跷的事情发生了！！

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116662.png)

利用点这里并没有canary

稍微翻了一下GCC的文档大概理解了

简单来说，GCC会用一种启发式的算法判断一个函数需不需要canary保护

如果需要则会生成

![img](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181106463.png)

当然也可以选择强制开启all-canary

最终payload很简单，因为这一层题目基本可以认为是没保护的

```
b'A'* 40 + p64(0x4017C0)
```

## 第二题touch2

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054785.png)

这里也比较基础

只需要pop edi;retn就行

先ROP找一下

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116687.png)

令0x000000000040141b为变量`val_pop_rdi`

同时因为没开PIE,所以用常数0x4017EC作为函数目标就可以了

画个图看看堆栈的样子

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116280.png)

这里的cookie是定值

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181100246.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054541.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116597.png)

在笔者的机器上，值为0x59b997fa

因此payload是

```
payload1 = b'A'* 40 + p64(val_pop_rdi) + p64(0x59b997fa) + p64(0x4017EC)
```

## 第三题touch3

这里太麻烦了(其实也没多麻烦,就是要搞个random,得再试验一下本机的random%100,而且考虑到前面已经操作过两三次random的样子,更麻烦了2333)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116362.png)

于是我打算直接劫持validate函数

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116905.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181054022.png)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172116512.png)

```
payload1 = b'A'* 40 + p64(val_pop_rbx) + p64(0x3) + p64(0x401D41)
```

总的来说还是比较简单的,因为没啥实质性保护

# 第二个文件rtarget

我写到这里才发现，这里第一个文件的预期解是代码注入，而我直接用ROP的方法做了（毕竟是pwn手）

但是实际上第二个文件的预期解才是ROP

所以这里我就不重复一遍了

（完）