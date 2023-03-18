---
title: C++ 循环链表解决约瑟夫环 leetcode 剑指Offer 62
poc: true
categories:
  - [笔记,课程,数据结构]
tags: []
id: '76'
date: 2021-04-22 13:54:48
---

很早之前就看过链表的一些基本思想和实现方法的伪代码，当时的第一感觉是挺简单的，实际写一遍之后出现不少细节上的坑点，这些细节大多不会在网上的文章当中细说，主要是debug笔记

最终正确的源码记录如下

```
#include <iostream>

using namespace std;


struct node {

    int data;

    node *next;

    node(int d, node *n = NULL) : data(d), next(n) {}

};

class Joseph {

private:

    node *head;

public:

    Joseph(int n);

    ~Joseph();

    void simulate();

};

Joseph::Joseph(int n) {
    if (n) {

        head = new node(1);
        node *temp = head;
        for (int i = 2; i <= n; ++i) {
            temp->next = new node(i);
            temp = temp->next;
        }
        temp->next = head;
    } else {
        head = new node(1);
        head->data = 0;
    }
}

Joseph::~Joseph() {

}

void Joseph::simulate() {
    int counter = 0;
    node *targetNode = head;
    node *temp;
    if (targetNode->next == targetNode) {
        cout << targetNode->data;
        delete targetNode;
        targetNode = NULL;
        return;
    }
    if (!targetNode->data) {
        cout << "No one!";
        delete targetNode;
        targetNode = NULL;
        return;
    }
    if (targetNode->next != targetNode) counter = 1;
    while (targetNode->next != targetNode) {
        targetNode = targetNode->next;
        temp = targetNode->next;
        cout << temp->data << " ";
        targetNode->next = temp->next;
        targetNode = temp->next;
        delete temp;
        temp = NULL;
    }
    if (counter) {
        cout << endl << targetNode->data;
        delete targetNode;
        targetNode = NULL;
        return;
    } else {
        cout << "No one!";
        delete temp;
        temp = NULL;
        return;
    }
}

int main() {
    int n;
    cin >> n;
    Joseph jos(n);
    jos.simulate();
    return 0;
}
```

### 构造函数

最初我的想法比较傻：输入 人数n，就要new N个节点，也就需要分配N个指针指向这些N个节点，但是问题是N的数量在编译阶段是不确定的，那么我该怎么命名这些指针`p1`,`p2`,`p3`....？毕竟这些指针的变量名需要在代码中预先声明，我甚至一度想要用对象数组或者map

后来才反应过来，我需求的这个所谓的指针，就是各个节点当中的成员变量`next`

因此我只需要先判断参数N是否合法,如果合法,就先给`head`指针`new`一个node对象,然后再给`head->next`指针`new`一个node对象for循环一遍,出口条件设置为`不大于`,退出循环后,再把最后一个node的next指向head

### simulate函数

这个函数我的最初实现方式,需要开辟两份空间，总觉得很傻，经过高人指点改成了现在的样子，现在放上一份我最初的代码，以及当时这么写的理由

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181056791.png)

假设当前targetNode指向了node(2),接下来的操作是（不分先后）:

1.  删除node(3)
2.  node(2)->next指向node(4)
3.  targetNode指向node(4)

这些操作肯定要有个先后顺序，最先可以确定的是`删除node(3)`是最后一个操作，因为该操作是释放内存，又因为对node(3)的唯一引用方式是`node(2)->next`，因此我们需要保证在最后一步时仍能找到node(2)，而不会出现野指针

同时，从代码复用的角度来考虑，引用node(2)的唯一方式是当targetNode指向node(2)时，使用targetNode指针进行引用

这样的话，我们进行`targetNode指向node(4)`这步操作时必须先用一个`temp1`指针保存

同时，在delete node(3)之前必须用一个指针保存`node(3)->next`否则会出现野指针

综上，需要两个临时变量

这样感觉实在是太傻了，总共才三个元素，居然要两个临时变量

更高效的解决办法有两个：

1.  双向链表
2.  文章最开始时那堆代码当中的实现方式

该方法通过只定义一个指针temp，获得了两个对象的操作权限：`node(3)`和`node(3)->next`

这种思维方式让我大受启发，我开始用这一方式继续写下去

但由于夹杂了`node(2)->next->next`这种写法，出现了0xdd的bug（第二轮simulate时delete时会越界 double free），后续debug时，类似的写法全都改写，改写原则为：对next指针的使用只限于一层的深度，如果要使用两层深度的next，则应当改为使用temp的next指针——这样只会调用一层深度的指针

这样，主体问题就都解决了，后续发现测试数据里面有`0`,`-1`等非法值，因此加入了几个if作为检测，每个if检测都是出口分支，所以需要加上`return;` 此外，还需要在`exit();`之前delete head节点和/或targetNode节点,同时将其置为`NULL`