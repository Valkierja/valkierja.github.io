---
title: 重写(override)和重载(overload),extern和virtual
poc: true
categories:
  - [笔记, 存档]
tags: [cpp]
id: '217'
date: 2021-07-19 00:05:12
---

看书(unity3D脚本编程,黄皮书)看到一句话有点在意

> Mono Runtime 总是知道各个对象的类型是什么,例如调用gettype方法便可获得对象的具体类型。而且由于gettype是一个实例方法，因而一个类型不可能伪装成另一个类型……Alin类不能通过重写gettype方法来返回一个Singer类

笔者高度怀疑这本书的作者，要是让她现在看这段话，他自己也看不懂这段话的意思

逻辑关系混乱，到底是**不能**还是**可以但没用**,或者换句话来说，Mono Runtime识别一个实例的类型的方法是什么，是调用gettype吗，还是其他专用方法？

经过笔者亲自反汇编相关库，发现如下内容

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181053625.png)

此外在StackOverflow上面查到了一句话

> C# 重写函数的前提是这个函数是一个虚函数

但在笔者印象里，C++类继承重写时，可以是虚函数也可以不是虚函数

lab后记录如下

在C++中基类方法不标记虚函数,派生类不标记override,仍可进行重写,基类方法与派生类当中的重名方法均可被正常调用,但IDE会提示C-Tidy

如果是在C#当中,要想重写方法,就必须保证基类方法是virtual的,且为非extern的

不过,如果是重载函数的话,则没有以上的限制,事实上,重载函数几乎没有什么限制

另外记录一个C++中,派生类调用基类的同名方法的办法

类A有方法foo

派生类B又方法foo

b是B的实例,则`b.A::foo()`即可