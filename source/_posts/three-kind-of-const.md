---
title: 三种const
poc: true
categories:
  - [开发,C++]
  - [笔记,短笔记]
tags: [悬而未决, 需要修改,cpp]
id: '153'
date: 2021-07-03 13:02:48
---

# 前言

本文主要讲的是const和template

# 正文

### 模板的重载

```
#include <iostream>
#include <vector>
#include <string>
using namespace std;

class students{
public:
    void printID(){
        cout<<ID;
    }
    void printName(){
        cout<<name;
    }
    int getID() const{
        return ID;
    }
private:
    int ID;
    string name;
};



template<typename T> T const& Max(T const& a,T const& b){
    return a<b?b:a;
}
template<typename T> int  Max(students const& a,students const& b){
    return a.getID()<b.getID()?b.getID():a.getID();
}

```

### 三种const

```
#include <iostream>
#include <vector>
#include <string>
using namespace std;

class students{
public:
    void printID(){
        cout<<ID;
    }
    void printName(){
        cout<<name;
    }
    int getID() const{
        return ID;
    }
private:
    int ID;
    string name;
};



template<typename T> T const& Max(T const& a,T const& b){
    return a<b?b:a;
}
template<typename T> int  Max(students const& a,students const& b){
    return a.getID()<b.getID()?b.getID():a.getID();
}
```

来看下面这个声明

`Student const& Choose(Student const& a1,Student const& a2 ) const;`

1.  `Student const& Choose` 中 const含义为返回值不可被变更,且返回值只能被赋值给同样被const修饰的指针
2.  参数当中的const代表这个按引用传递的参数的值在整个函数体当中不可被更改
3.  函数声明最末尾的const表明这整个函数是只读函数(read-only),这个标记符会帮助编译器检查函数体当中有没有修改任何成员变量的操作,如果有则会报错,同时这个标记符还会帮助其他程序员理解这个函数的功能