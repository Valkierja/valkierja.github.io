---
title: 二进制杂记 lea、内存对齐、小端的一些注意点
poc: true
categories:
  - - CTF
  - - '二进制'
    - PWN
    - RE
tags: []
id: '302'
date: 2021-08-12 02:06:54
---

lea eax,\[ebx+8\]就是将ebx+8这个值直接赋给eax，而不是把ebx+8处的内存地址里的数据赋给eax。  
mov eax,\[ebx+8\]则是把内存地址为ebx+8处的数据赋给eax

内存被CPU读取时以dw为对齐，8的倍数

小端的一个注意点来自于C语言中文网的一个lab的反汇编，誊抄如下

```
void makeArray()
{
    char myString[30];
    for ( int i = 0; i < 30; i++ )
        myString[i] = '*';
}

---------------------------------------------------------------

makeArray PROC
    push ebp
    mov ebp,esp
    sub esp, 32            ;myString 位于 EBP-30 的位置
    lea esi, [ebp-30]      ;加载 myString 的地址
    mov ecx, 30            ;循环计数器
LI: mov BYTE PTR [esi]     ;填充一个位置
    inc esi                ;指向下一个元素
    loop LI                ;循环，直到 ECX=0
    add esp, 32            ;删除数组(恢复ESP)
    pop ebp
    ret
makeArray ENDP
```

首先，栈增长方向向下，根据小端序，低位数据放在低位，所以str的第一个元素几乎在栈顶，但由于对齐的原因，栈实际增长为32

另外有如下代码的栈空间示例

```
void foo(){
int a[5];
int b[3];
return;
}
```

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181056110.png)

也就是说，相邻数组下越界发生的时不是读取到了下一个数组的第一个元素，而是下一个数组的最后一个元素

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303181055415.png)

另外IDA这个小端数据显示方式是

54h是最低位的,存在地址低位,也就是左边的起始地址

同时地址最低位存放的是数组角标最小的元素

地址最高位存放的是角标最大的元素