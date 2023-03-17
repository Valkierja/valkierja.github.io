---
title: webbench源码分析与程序改进及SOCKS网络协议笔记 （2）什么是socks，Linux平台面向socks编程笔记
poc: true
categories:
  - - CTF
tags: []
id: '578'
date: 2021-08-29 18:44:50
---

在单端多进程，IP池有限等因素的加持下，出现了socks协议，本质上来说，假如目标IP为192.168.1.2，端口为88，则指针“192.168.1.2:88”所指向的目标，就是一个socks对象实例

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172115498.png)

socks的本义其实是网络数据线末端的那个卡扣

另外补一个数据边界的笔记

SOCK\_STREAM是无边界的，SOCK\_DGRAM是有边界的

这种边界并非体现在底层硬件的MTU大小和切片分发，分片传输机制，而是体现在通信双方应用层面，在这个问题上，我们要把硬件层看成透明的

然后如何理解边界一词，也就是数据是连续的，接受和发送是异步的，读取和发送可以是断续的，可以在buf满时操作，也可以在未满时操作，双方的行动不需要统一，虽然内容可以是断续的，但连接是流状的，是连续存在的

而有数据边界则恰恰相反，收发的连接双方在线程操作buf的次数和相对行为上必须同步，同时收发包的顺序无法做到相同

## 面向socks协议栈开发

代码中出现了一堆底层类和结构体，正好学一下TCP协议，之前草草学过一点点，基本就只知道了个DHCP。。。

sockaddr\_in结构体

```
struct sockaddr_in {
               sa_family_t    sin_family; /* address family: AF_INET */ 指明协议（实际上这个结构体是专门用于IPv4的）
               in_port_t      sin_port;   /* port in network byte order */指明端口
               struct in_addr sin_addr;   /* internet address */指明地址

               //某些版本的源码中,此处另有八个字节的零位对齐
           };


           struct in_addr {
               uint32_t       s_addr;     /* address in network byte order */只能IPv4
           };
```

文档如下

![](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202303172115948.png)

hostent结构体

```
struct hostent {
char *h_name; //域名字符串
char **h_aliases;  //别名字符串
int h_addrtype;//IPv4
int h_length;
char **h_addr_list;
// 网络字节序的IP地址
};
```

写完注释的fSocks函数如下

```
int fSocket(const char *host, int clientPort)
{
    unsigned int sock;
    unsigned long inaddr;//二进制形式地址
    struct sockaddr_in ad;//存放地址
    struct hostent *hp;
    
    memset(&ad, 0, sizeof(ad)); //分配内存
    ad.sin_family = AF_INET;  //IPv4

    inaddr = inet_addr(host);//将地址字符串转二进制形式的地址
    if (inaddr != INADDR_NONE)//INADDR_NONE报错,如果输入非法IP字符串255 IP,都会出现该报错
        memcpy(&ad.sin_addr, &inaddr, sizeof(inaddr));
    else
    {
        hp = gethostbyname(host);//接收到exception为INADDR_NONE的IP,尝试解析该IP
        if (hp == NULL)//无法解析IP,证明IP错误
            return -1;
        memcpy(&ad.sin_addr, hp->h_addr, hp->h_length);//可以向DNS解析IP,IP正确
    }
    ad.sin_port = htons(clientPort);//把int型转换为网络传输字节型
    
    sock = socket(AF_INET, SOCK_STREAM, 0);//建立IPv4,流格式,TCP数据连接,获取连接FD
    if (connect(sock, (struct sockaddr *)&ad, sizeof(ad)) < 0)//double check Link exist
        return -1;
    return sock;
}
```

之后就是主体部分了