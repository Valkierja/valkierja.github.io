---
title: 枚举,结构体,yield return与多线程通信
poc: true
categories:
  - [开发, GamePlay]
tags: []
id: '203'
date: 2021-07-16 14:23:25
---

## 背景

Unity开发角色死亡后过一段时间复活,但是有如下问题

角色死亡后`setActive(false)`后,角色所属组件(包括PlayerController.cs)都将不再执行,此时无法通过脚本拉起角色复活函数;解决方法有两个

第一个比较傻,但也简单:设置一个透明空类在地图的犄角旮旯里面,持续监听player状态,在角色死亡后,复活CD结束时拉起角色复活函数

第二个是业界常规做法,使用枚举器+yield return新建一个独立于Update函数的线程,监听已死亡的角色状态

## 正文

先附上一段Unity文档,这个文档解决了我的许多疑问[https://docs.unity3d.com/Manual/Coroutines.html](https://docs.unity3d.com/Manual/Coroutines.html)

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181049147.png)

简单来说,枚举迭代被用于多线程通信

对于一般的C#脚本来说,主要的线程用于持续调用Update函数

我们需要使用yield return配合函数声明为IEnumerator返回值

```
IEnumerator foo(){

yield return result;
}
```

所以只需要

```
    void Update()
    {
        
        if (!diamond.activeInHierarchy)
        {
            spawnDiamondCo();
        }
    }

    public void  spawnDiamondCo()
    {
        StartCoroutine("spawnDiamond");
    }

 public IEnumerator spawnDiamond()
    {
        yield return new WaitForSeconds(3);
        randomX = Random.Range(-5, 5);
        randomY = Random.Range(-4.5f,4.5f);
        pos = new Vector3(randomX,0.0f , randomY);
        diamond=Instantiate(diamondPrefab,pos,Quaternion.identity);
        StopCoroutine("spawnDiamond");
    }

```

别忘了 StopCoroutine("spawnDiamond")

另外还有一种情况,需要一个协程A拉起另一个协程B再由B拉起C

此时如何正确退出

两种办法

第一种FILO

第二种直接结束最高级的父协程,测试下来没啥问题