---
title: 网络字节序与主机字节序
comments: true
date: 2020-01-12 14:48:02
updated: 2020-01-12 14:48:02
tags:
categories: Network
permalink:
---

# 主机字节序
在网络编程中涉及到的IP地址和端口号通常是多字节整数，多字节整数在不同的硬件结构上字节序可能不同。所谓字节序指的是对于存储需要多个字节（大于 1 字节）的整数来说，其每个字节在不同的机器内存中存储的顺序。这就是所谓的`主机字节序`，一般分为两类：

## 大端序(big endian)
内存地址从小到大，先存储多字节整数的最高有效位，称之为大端序。

## 小端序(little endian)
内存地址从小到大，先存储多字节整数的最低有效位，称之为小端序。

下图为ox1234567用不同的字节序存储时的示意图
![](/images/big-little-endian.gif)

# 网络字节序
为了在信息传输时，屏蔽掉不同硬件结构上的字节序的差异，TCP/IP协议规定，所有在网络上传输的多字节整数都以大端序编码，所以大端序就是`网络字节序`.

# 主机字节序与网络字节序之间的转换
``` c
#include <arpa/inet.h>
//host to net long
uint32_t htonl(uint32_t hostlong);
//host to net short
uint16_t htons(uint16_t hostshort);

//net to host long
uint32_t ntohl(uint32_t netlong);
//net to host short
uint16_t ntohs(uint16_t netshort);
```
再讲主机上的多字节整数发送到网络中时，必须将主机字节序转换到网络字节序。从网络中接收到多字节整数时也要讲网络字节序转换为主机字节序。当然如果主机字节序本来就是大端序，那么转换函数就什么都没做。
