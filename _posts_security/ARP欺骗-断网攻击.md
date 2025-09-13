---
title: ARP攻击
date: 2021-02-17 12:03:04
tags:
categories:
    - 网络安全
toc: true
# keywords:
description:
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
#theme: default
#password: "liuzhenlong"
---

# 1 理论

## 1.1 原理

<img src="https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/Snipaste_2021-02-18_14-20-42.jpg" width = "600" height = "400" alt="图片名称" align=center id=25 />

　　ARP攻击主要是通过伪造IP地址和MAC地址进行欺骗。假设A、B、C位于`同一局域网`内，A是PC主机，B是攻击主机，C是路由器。攻击过程如下：

1. A将包含目标IP地址(10.0.1.1)信息的ARP Request，广播到网络中的所有主机，源主机A的IP地址和MAC地址都包括在ARP请求中；
2. C确定ARP请求中的IP地址与自己的IP地址匹配，则将主机A的IP地址和MAC地址[映射](https://baike.baidu.com/item/映射)添加到本地ARP缓存中；并将包含其MAC地址的ARP回复消息直接发送回主机A；
3. 当主机A收到从C发来的ARP回复消息时，会用C的IP和MAC地址映射更新ARP缓存；
4. 攻击主机B向A发送伪造的ARP应答包，在这个伪造的应答包中，IP地址为C的IP地址，而MAC地址为B的MAC地址；
5. A在接收到这个应答包后，会刷新它的ARP缓存，这样在A的ARP缓存表中就出现了IP-C与MAC-B的映射；
6. 攻击主机B向C发送伪造的ARP应答包，在这个伪造的应答包中，IP地址为A的IP地址，而MAC地址为B的MAC地址；
7. C在接收到这个应答包后，会刷新它的ARP缓存，这样在C的ARP缓存表中就出现了IP-A与MAC-B的映射。

　　这样一来，A与C之间的流量交换都会经过B，可以进行`监听、篡改信息、断网攻击`。

## 1.2 知识储备

* **网关**：网关(Gateway)在网络层以上实现网络互连。由于历史的原因，许多有关TCP/IP的文献曾经把网络层使用的路由器称为网关，在今天很多局域网采用路由来接入网络，因此通常指的网关就是路由器的IP！

* **Bridged(桥接模式)**：将主机网卡与虚拟机虚拟的网卡利用虚拟网桥进行通信。其网络结构如下图所示：

  <img src="https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/v2-0e29e8f13495c269f3fdc723aa2255c2_r.jpg" width = "800" height = "600" alt="图片名称" align=center id=26 />

# 2 实战

## 2.1 断网攻击

1. **准备**：`kali`虚拟机，设置桥接模式，否则断网无效；

2. **查看自己的网卡信息**

   <img src="https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/Snipaste_2021-02-17_22-40-47.jpg" width = "600" height = "300" alt="图片名称" align=center id=27 />

3. **找到目标IP**：`nmap`扫描内网(命令`nmap -sP 192.168.1.*`)或使用安卓端的`Net Analyzer`进行`LAN Scan`；

   <img src="https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/Snipaste_2021-02-17_22-04-15.jpg" width = "600" height = "300" alt="图片名称" align=center id=28 />

4. **arpspoof进行攻击**：执行下面命令后目的主机无法上网；

   ```shell
   arpspoof -i 自己网卡信息 -t 目标IP -r 网关
   例：arpspoof -i eth0  -t  192.168.1.7 -r 192.168.1.1
   ```

## 2.2 窃听

1. **开启IP转发**：`echo 1 >/proc/sys/net/ipv4/ip_forward`
2. **arp攻击**：`arpspoof -i eth0 -t targetIP -r gateway（具体参考上面）`
3. **窃听**：`driftnet -i eth0（你的网卡）`或`wireshark`

# 3 存在问题

* 攻击局域网其他系统时，kali虚拟机可以上网，但是其宿主机(物理机win10)无法上网；<font color=#008000>开启IP转发可以解决</font>
* 窃听时目标主机虽然可以ping通，当时网速太慢且不稳定；driftnet无法正确抓取图片；

# 4 参考

* [百度百科-ARP](https://baike.baidu.com/item/ARP/609343)
* [百度百科-ARP攻击](https://baike.baidu.com/item/ARP%E6%94%BB%E5%87%BB#:~:text=ARP%20%E7%97%85%E6%AF%92%E6%94%BB%E5%87%BB%E6%98%AF%E5%B1%80%E5%9F%9F%E7%BD%91,%E6%AD%A3%E5%B8%B8%E4%B8%8A%E7%BD%91%E5%92%8C%E9%80%9A%E4%BF%A1%E5%AE%89%E5%85%A8%E3%80%82)

* [百度百科-ARP欺骗](https://baike.baidu.com/item/ARP%E6%AC%BA%E9%AA%97)
* [百度百科-网关](https://baike.baidu.com/item/%E7%BD%91%E5%85%B3)
* [vmware 虚拟机三种网络模式—“桥接、NAT 、仅主机”区别？](https://zhuanlan.zhihu.com/p/56658358#:~:text=%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F%E5%B0%B1%E6%98%AF%E5%B0%86%E4%B8%BB%E6%9C%BA,%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AE%E8%80%8C%E4%B8%8D%E5%B9%B2%E6%89%B0%E3%80%82)

* [Kali Linux使用arpspoof命令进行断网攻击（ARP欺骗）【VMware一定要设置桥接模式，否则断网会无效】](https://blog.csdn.net/weixin_43343144/article/details/102679135)

* [使用kali进行ARP攻击与欺骗](https://blog.csdn.net/weixin_44839457/article/details/108333653?utm_source=app&app_version=4.5.2)
* [使用kali进行局域网ARP欺骗攻击与监听](https://blog.csdn.net/qq_42039946/article/details/112968942?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control)
* [秦柯-ARP攻击](https://www.youtube.com/watch?v=ko4VvZuO4rs)