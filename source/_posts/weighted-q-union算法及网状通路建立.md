---
title: weighted q-Union算法及网状通路建立
poc: true
categories:
  - [笔记,校内,数据结构]

tags: []
id: '1217'
date: 2021-10-14 09:23:41
---

假设我们用树解决这个问题

则我们会使用到单向树（由单链表衍生）

为了简化出核心思想，这里不使用地址的指向作为单链表，而是使用ID入口

也就是说我们维护ID数组，数组的下标为元素的ID，数组的内容为对应元素的父节点ID

通过这种方式回溯根节点

当我们想要引入权重时，我们只需要再引入并维护sz数组，sz数组的每一项含义为

下标作为节点序号，对应下标的内容为 该节点所在的整个树的节点个数

在合并前检查两树大小，小树在下，大树在上

然后这里有一个关于树的增长速率的简单说明

如果一个树有N层，那么他大致的总节点数量在2^N这个数量级

反过来说，如果一个树有N个节点，那么他的层数大致会有lgN这个数量级

如果同步增正N+a，那么lg（N+a），递推公式可证，引入权重的树的归并是lg（N）的时间复杂度

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181053606.png)