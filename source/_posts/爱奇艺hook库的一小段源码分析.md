---
title: 爱奇艺Hook库的一小段源码分析
poc: true
categories:
  - - CTF
  - - ctf
    - 二进制
tags: []
id: '1080'
date: 2021-09-15 16:50:18
---

## 背景

例子实在是太好了，必须分析一波

## 例题源码

```
#include <inttypes.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/mman.h>
#include <test.h>

#define PAGE_START(addr) ((addr) & PAGE_MASK)
#define PAGE_END(addr)   (PAGE_START(addr) + PAGE_SIZE)

void *my_malloc(size_t size)
{
    printf("%zu bytes memory are allocated by libtest.so\n", size);
    return malloc(size);
}

void hook()
{
    char       line[512];
    FILE      *fp;
    uintptr_t  base_addr = 0;
    uintptr_t  addr;

    //find base address of libtest.so
    if(NULL == (fp = fopen("/proc/self/maps", "r"))) return;
    while(fgets(line, sizeof(line), fp))
    {
        if(NULL != strstr(line, "libtest.so") &&
           sscanf(line, "%"PRIxPTR"-%*lx %*4s 00000000", &base_addr) == 1)
            break;
    }
    fclose(fp);
    if(0 == base_addr) return;

    //the absolute address
    addr = base_addr + 0x3f90;
    
    //add write permission
    mprotect((void *)PAGE_START(addr), PAGE_SIZE, PROT_READ  PROT_WRITE);

    //replace the function address
    *(void **)addr = my_malloc;

    //clear instruction cache
    __builtin___clear_cache((void *)PAGE_START(addr), (void *)PAGE_END(addr));
}

int main()
{
    hook();
    
    say_hello();
    return 0;
}
```

主要看第十一行和第四十二行

## 正文

第十一行是一个返回值为空指针的函数，这里目前还不太知道为什么，因为如果改成void是不影响后面操作的，姑且先认为是原作者的编写习惯

第四十二行 是一个改变指针解读的操作，很精妙，也很基础

主题逻辑是：

1.  读入函数头表并定位基址
2.  改变页权限
3.  改写函数地址指向
4.  清除指令地址缓存\_\_builtin\_\_\_clear\_cache，迫使CPU重新定位
5.  返回