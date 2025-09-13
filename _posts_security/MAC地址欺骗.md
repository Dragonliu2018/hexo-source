---
title: MAC地址泛洪与欺骗
date: 2021-03-04 21:47:39
tags:
categories:
    - [网络安全]
# keywords:
description:
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/下载1.png
toc: true #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
#theme: default
#password: "liuzhenlong"
---

# 1 理论

## 1.1 原理

　　MAC地址欺骗是数据链路层攻击，它是利用**交换机端口学习的漏洞**，通过客户端向交换机发送欺骗报文、攻击交换机的CAM表的方式，使交换机CAM表的记录与真实的主机对应MAC地址不一致，从而使交换机将报文错误转发给攻击者。

　　MAC地址泛洪攻击，每台交换机的CAM表都有最大记录条数，当交换机端口学习得到的记录大于交换机CAM表条数上限时，交换机收到一个网络帧，只要其目标主机的MAC地址不存在于该交换机CAM表中，那么该帧会以广播的方式发向交换机的每一个端口，此时的交换机就相当于一个集线器HUB，我们就可以监听网络中的所有通讯报文。

　　与ARP攻击不同：MAC地址欺骗改变的是端口/MAC地址映射关系，ARP攻击改变的是IP/MAC地址映射关系。

　　MAC地址泛洪攻击针对路由器不行，原因是路由器仅仅转发特定地址的数据包，不传送不支持路由协议的数据包传送和未知目标网络数据包的传送，从而可以防止广播风暴。参考：[交换机和路由器的区别在哪里 一针见血通俗解答](https://www.cnblogs.com/Lynn-Zhang/articles/5754336.html)

　　实验网络拓扑如下：

<img src="https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/Snipaste_2021-03-05_11-19-37.jpg" width = "700" height = "400" alt="图片名称" align=center id=32 />

## 1.2 知识储备

* **MAC地址**：它是一个用来确认网络设备位置的位址。在[OSI模型](https://baike.baidu.com/item/OSI模型)中，第三层网络层负责[IP地址](https://baike.baidu.com/item/IP地址)，第二层数据链路层则负责MAC位址。MAC地址用于在网络中唯一标示一个[网卡](https://baike.baidu.com/item/网卡)，一台设备若有一或多个网卡，则每个网卡都需要并会有一个唯一的MAC地址。由网络设备制造商生产时烧录在网卡(Network lnterface Card)的[EPROM](https://baike.baidu.com/item/EPROM/1690813)(一种闪存芯片，通常可以通过程序擦写)。[IP地址](https://baike.baidu.com/item/IP地址/150859)与MAC地址在计算机里都是以[二进制](https://baike.baidu.com/item/二进制/361457)表示的，IP地址是32位的，而MAC地址则是48位的，通常表示为12个16进制数，如：00-16-EA-AE-3C-40就是一个MAC地址。
* **交换机**：主要功能是在局域网内的主机之间转发数据帧，工作核心是**端口/MAC地址映射表**，这张表里记录了交换机每个端口同与之相连的主机MAC地址之间的对应关系。交换机接收从主机上传来的比特流之后，要先将其缓存成数据帧，然后通过分析帧头中的MAC地址，交换机就可以判断出数据的发送方和接收方，在映射表里查找与数据帧的目的MAC地址相对应的端口。若找到则发给目的主机；若找不到则以广播的方式将这个数据帧向除了源端口以外的其它所有端口转发，从而将数据发送给了目的主机，并记录端口与MAC地址的对应关系。

# 2 实战

## 2.1 MAC地址欺骗

1. **准备**：没有交换机物理设备，vmware中桥接模式中有虚拟交换机(但是无法登陆后台)，所以只能利用路由器测试(也无法进入后台)，只能查看现象了。
2. 此处未进行实践，可参考：[MAC地址欺骗与MAC地址泛洪攻击（eNSP环境演示）](https://blog.csdn.net/redwand/article/details/105388491?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-0&spm=1001.2101.3001.4242)

## 2.2 MAC地址泛洪攻击

1. 以`kali`为攻击机，使用`macof`命令向网络中发送大量网络不存在的源mac地址，堵塞交换机(路由器)的CAM表；
2. 攻击前，`Win物理机`可以ping通kali、node01、node02、node03；
3. 攻击后，按道理此时的路由器中的CAM表中存在大量陌生MAC地址记录，但是目前无法查看；现象为：请求超时或间断ping通，但可以ping通外网及局域网内其它主机。

# 3 存在问题

* 没有物理交换机，虚拟交换机无法登陆后台
* 用路由器测试，但是无法登录后台查看CAM表，可参考：[小米路由器开启SSH](https://cloud.tencent.com/developer/article/1482014)

# 4 参考

* [百度百科-MAC地址](https://baike.baidu.com/item/MAC%E5%9C%B0%E5%9D%80/1254181?fr=aladdin)
* [MAC泛洪***的实施与防御](https://blog.csdn.net/weixin_34234829/article/details/85088965?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.baidujs&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.baidujs)
* [MAC地址欺骗与MAC地址泛洪攻击（eNSP环境演示）](https://blog.csdn.net/redwand/article/details/105388491?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-0&spm=1001.2101.3001.4242)
* [亁颐堂现任明教教主CCNA Security 2019 7 第三天 2 MAC泛洪MAC欺骗](https://www.youtube.com/watch?v=FiDq8g0gBsI)