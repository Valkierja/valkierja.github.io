---
title: 转爱奇艺的Hook教程
poc: true
categories:
  - - CTF
  - - CTF
    - [信息安全, 二进制]
tags: []
id: '1071'
date: 2021-09-15 16:13:40
---

原文根据BSD开源，所有权利归爱奇艺所有

https://github.com/iqiyi/xHook

编者感觉很多部分的内容写的很好，直接复制了一部分又补充了一部分，充当笔记

# 原文摘选

静态ELF使用section 组织各个部分，而SHT就是记录各个section的表，主要包括：section 的类型、在文件中的偏移量、大小、加载到内存后的虚拟内存相对地址、内存中字节的对齐方式等。

SHT中，与Hook相关的几个节区是

*   `.dynstr`：保存了所有的字符串常量信息。
*   `.dynsym`：保存了符号（symbol）的信息（符号的类型、起始地址、大小、符号名称在 `.dynstr` 中的索引编号等）。函数也是一种符号。
*   `.text`：程序代码经过编译后生成的机器指令。
*   `.dynamic`：供动态链接器使用的各项信息，记录了当前 ELF 的外部依赖，以及其他各个重要 section 的起始位置等信息。
*   `.got`：Global Offset Table。用于记录外部调用的入口地址。动态链接器（linker）执行重定位（relocate）操作时，这里会被填入真实的外部调用的绝对地址。
*   `.plt`：Procedure Linkage Table。外部调用的跳板，主要用于支持 lazy binding 方式的外部调用重定位。（Android 目前只有 MIPS 架构支持 lazy binding）
*   `.rel.plt`：对外部函数直接调用的重定位信息。
*   `.rel.dyn`：除 `.rel.plt` 以外的重定位信息。（比如通过全局函数指针来调用外部函数）

![](https://raw.githubusercontent.com/iqiyi/xHook/master/docs/overview/res/elfpltgot.png)

在ELF加载到内存时，各个部分使用segment 组织，一个 segment 包含了单/复数个 section。ELF 使用 PHT 来记录所有 segment 的基本信息。主要包括：segment 的类型、在文件中的偏移量、大小、加载到内存后的虚拟内存相对地址、内存中字节的对齐方式等。

所有类型为 `PT_LOAD` 的 segment 都会被动态链接器（linker）映射（mmap）到内存中。

例：libtest.so 的 PHT：

```
caikelun@debian:~$ arm-linux-androideabi-readelf -l ./libtest.so 

Elf file type is DYN (Shared object file)
Entry point 0x0
There are 8 program headers, starting at offset 52

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  PHDR           0x000034 0x00000034 0x00000034 0x00100 0x00100 R   0x4
  LOAD           0x000000 0x00000000 0x00000000 0x02604 0x02604 R E 0x1000
  LOAD           0x002e3c 0x00003e3c 0x00003e3c 0x001c8 0x001c8 RW  0x1000
  DYNAMIC        0x002e48 0x00003e48 0x00003e48 0x00118 0x00118 RW  0x4
  NOTE           0x000134 0x00000134 0x00000134 0x000bc 0x000bc R   0x4
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x10
  EXIDX          0x002504 0x00002504 0x00002504 0x00100 0x00100 R   0x4
  GNU_RELRO      0x002e3c 0x00003e3c 0x00003e3c 0x001c4 0x001c4 RW  0x4

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .note.android.ident .note.gnu.build-id .dynsym .dynstr .hash .gnu.version .gnu.version_d .gnu.version_r .rel.dyn .rel.plt .plt .text .ARM.extab .ARM.exidx 
   02     .fini_array .init_array .dynamic .got .data 
   03     .dynamic 
   04     .note.android.ident .note.gnu.build-id 
   05     
   06     .ARM.exidx 
   07     .fini_array .init_array .dynamic .got
```

## 连接视图（Linking View）和执行视图（Execution View）

*   连接视图：ELF 未被加载到内存执行前，以 section 为单位的数据组织形式。
*   执行视图：ELF 被加载到内存后，以 segment 为单位的数据组织形式。

我们关心的 hook 操作，属于动态形式的内存操作，因此主要关心的是执行视图，即 ELF 被加载到内存后，ELF 中的数据是如何组织和存放的。

![](https://raw.githubusercontent.com/iqiyi/xHook/master/docs/overview/res/elfview.png)

## .dynamic section

这是一个十分重要和特殊的 section，其中包含了 ELF 中其他各个 section 的内存位置等信息。在执行视图中，总是会存在一个类型为 `PT_DYNAMIC` 的 segment，这个 segment 就包含了 .dynamic section 的内容。

无论是执行 hook 操作时，还是动态链接器执行动态链接时，都需要通过 `PT_DYNAMIC` segment 来找到 .dynamic section 的内存位置，再进一步读取其他各项 section 的信息。

libtest.so 的 .dynamic section：

```
caikelun@debian:~$ arm-linux-androideabi-readelf -d ./libtest.so 

Dynamic section at offset 0x2e48 contains 30 entries:
  Tag        Type                         Name/Value
 0x00000003 (PLTGOT)                     0x3f7c
 0x00000002 (PLTRELSZ)                   240 (bytes)
 0x00000017 (JMPREL)                     0xcb8
 0x00000014 (PLTREL)                     REL
 0x00000011 (REL)                        0xc78
 0x00000012 (RELSZ)                      64 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x6ffffffa (RELCOUNT)                   3
 0x00000006 (SYMTAB)                     0x1f0
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000005 (STRTAB)                     0x590
 0x0000000a (STRSZ)                      1201 (bytes)
 0x00000004 (HASH)                       0xa44
 0x00000001 (NEEDED)                     Shared library: [libc.so]
 0x00000001 (NEEDED)                     Shared library: [libm.so]
 0x00000001 (NEEDED)                     Shared library: [libstdc++.so]
 0x00000001 (NEEDED)                     Shared library: [libdl.so]
 0x0000000e (SONAME)                     Library soname: [libtest.so]
 0x0000001a (FINI_ARRAY)                 0x3e3c
 0x0000001c (FINI_ARRAYSZ)               8 (bytes)
 0x00000019 (INIT_ARRAY)                 0x3e44
 0x0000001b (INIT_ARRAYSZ)               4 (bytes)
 0x0000001e (FLAGS)                      BIND_NOW
 0x6ffffffb (FLAGS_1)                    Flags: NOW
 0x6ffffff0 (VERSYM)                     0xbc8
 0x6ffffffc (VERDEF)                     0xc3c
 0x6ffffffd (VERDEFNUM)                  1
 0x6ffffffe (VERNEED)                    0xc58
 0x6fffffff (VERNEEDNUM)                 1
 0x00000000 (NULL)                       0x0
```

## [](https://github.com/iqiyi/xHook/blob/master/docs/overview/android_plt_hook_overview.zh-CN.md#%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%99%A8linker)动态链接器（linker）

动态链接（比如执行 dlopen）的大致步骤是：

1.  检查已加载的 ELF 列表。（如果 libtest.so 已经加载，就不再重复加载了，仅把 libtest.so 的引用计数加一，然后直接返回。）
2.  从 libtest.so 的 .dynamic section 中读取 libtest.so 的外部依赖的 ELF 列表，从此列表中剔除已加载的 ELF，最后得到本次需要加载的 ELF 完整列表（包括 libtest.so 自身）。
3.  逐个加载列表中的 ELF。加载步骤：
    *   用 `mmap` 预留一块足够大的内存，用于后续映射 ELF。（`MAP_PRIVATE` 方式）
    *   读 ELF 的 PHT，用 `mmap` 把所有类型为 `PT_LOAD` 的 segment 依次映射到内存中。
    *   从 .dynamic segment 中读取各信息项，主要是各个 section 的虚拟内存相对地址，然后计算并保存各个 section 的虚拟内存绝对地址。
    *   执行重定位操作（relocate），这是最关键的一步。重定位信息可能存在于下面的一个或多个 secion 中：`.rel.plt`, `.rela.plt`, `.rel.dyn`, `.rela.dyn`, `.rel.android`, `.rela.android`。动态链接器需要逐个处理这些 `.relxxx` section 中的重定位诉求。根据已加载的 ELF 的信息，动态链接器查找所需符号的地址（比如 libtest.so 的符号 `malloc`），找到后，将地址值填入 `.relxxx` 中指明的**目标地址**中，这些“**目标地址**”一般存在于`.got` 或 `.data` 中。
    *   ELF 的引用计数加一。
4.  逐个调用列表中 ELF 的构造函数（constructor），这些构造函数的地址是之前从 .dynamic segment 中读取到的（类型为 `DT_INIT` 和 `DT_INIT_ARRAY`）。各 ELF 的构造函数是按照依赖关系逐层调用的，先调用被依赖 ELF 的构造函数，最后调用 libtest.so 自己的构造函数。（ELF 也可以定义自己的析构函数（destructor），在 ELF 被 unload 的时候会被自动调用）

## 例子

#include <inttypes.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/mman.h>
#include <test.h>

#define PAGE\_START(addr) ((addr) & PAGE\_MASK)
#define PAGE\_END(addr)   (PAGE\_START(addr) + PAGE\_SIZE)

void \*my\_malloc(size\_t size)
{
    printf("%zu bytes memory are allocated by libtest.so\\n", size);
    return malloc(size);
}

void hook()
{
    char       line\[512\];
    FILE      \*fp;
    uintptr\_t  base\_addr = 0;
    uintptr\_t  addr;

    //find base address of libtest.so
    if(NULL == (fp = fopen("/proc/self/maps", "r"))) return;
    while(fgets(line, sizeof(line), fp))
    {
        if(NULL != strstr(line, "libtest.so") &&
           sscanf(line, "%"PRIxPTR"-%\*lx %\*4s 00000000", &base\_addr) == 1)
            break;
    }
    fclose(fp);
    if(0 == base\_addr) return;

    //the absolute address
    addr = base\_addr + 0x3f90;
    
    //add write permission
    mprotect((void \*)PAGE\_START(addr), PAGE\_SIZE, PROT\_READ  PROT\_WRITE);

    //replace the function address
    \*(void \*\*)addr = my\_malloc;

    //clear instruction cache
    \_\_builtin\_\_\_clear\_cache((void \*)PAGE\_START(addr), (void \*)PAGE\_END(addr));
}

int main()
{
    hook();
    
    say\_hello();
    return 0;
}