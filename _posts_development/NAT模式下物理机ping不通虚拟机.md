---
title: NAT模式下物理机ping不通虚拟机
tags: 
categories:
    - [环境与工具]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-28 16:10:09
---

# 1 情景描述

* 物理机：Windows10
* 虚拟机：Ubuntu20.04 LTS
* 网络模式：NAT模式
* 物理机IP：192.168.31.108
* 虚拟机IP：192.168.10.138

虚拟机可以Ping通物理机：

```sh
➜  ~ ping 192.168.31.108
PING 192.168.31.108 (192.168.31.108) 56(84) bytes of data.
64 bytes from 192.168.31.108: icmp_seq=1 ttl=128 time=0.838 ms
64 bytes from 192.168.31.108: icmp_seq=2 ttl=128 time=0.415 ms
```

物理机Ping不通虚拟机：

```sh
E:\01 Blog_work\Hexo-Blog>ping 192.168.10.138

正在 Ping 192.168.10.138 具有 32 字节的数据:
请求超时。
请求超时。
```

# 2 问题原因

 VMnet8(`192.168.137.1`)与虚拟机IP(`192.168.10.138`)不在一个网段上：

```sh
E:\01 Blog_work\Hexo-Blog>ipconfig

Windows IP 配置
...
以太网适配器 VMware Network Adapter VMnet8:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::5fb:8fe:8535:fd2%23
   IPv4 地址 . . . . . . . . . . . . : 192.168.137.1
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . :
```

# 3 解决

修改网卡(`VMware Virtual Ethernet Adapter for VMnet8`)的配置信息：

<img src="https://s2.loli.net/2022/02/28/9YzxsWEXkC7ydR1.png" width = "800" height = "600" alt="图片名称" align=center id=98 />

禁用VM8网卡，然后再启用就可以了：

```sh
E:\01 Blog_work\Hexo-Blog>ping 192.168.10.138

正在 Ping 192.168.10.138 具有 32 字节的数据:
来自 192.168.10.138 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.10.138 的回复: 字节=32 时间<1ms TTL=64
```

# X 参考

* [解决NAT模式下物理机ping不通虚拟机](https://blog.csdn.net/qq_43200143/article/details/113463549)
