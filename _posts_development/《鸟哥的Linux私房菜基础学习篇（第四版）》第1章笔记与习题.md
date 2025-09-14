---
title: 《鸟哥的Linux私房菜基础学习篇（第四版）》第1章笔记与习题
tags:
categories:
  - 阅读
  - [操作系统, Linux]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-24 10:49:40
---

> 第1章：Linux是什么与如何学习

# 1 笔记

> 另外，不要再说没兴趣了。没有花时间去了解一下，不要跟人家说你没兴趣！而且，兴趣也是靠培养来的！除了某些特殊人物之外， 没有花时间去培养兴趣，怎么可能会有兴趣。

# 2 习题

## 1.1 你在你的主机上面安装了一张网络卡，但是开机之后，系统却无法使用，你确定网络卡是好的，那么可能的问题出在哪里？该如何解决？

因为所有的硬件都没有问题，所以，可能出问题的地方在于系统的核心(kernel) 不支持这张网络卡。解决的方法：

1. 到网络卡的开发商网站
2. 下载支持你主机操作系统的驱动程序
3. 安装网卡驱动程序后，就可以使用了。

## 1.2 一个操作系统至少要能够完整的控制整个硬件，请问，操作系统应该要控制硬件的哪些单元？

根据硬件的运作，以及数据在主机上面的运算情况与写入/读取情况，我们知道至少要能够控制：

1. input/output control
2. device control
3. process management
4. file management等等

## 1.3 我在 Windows 上面玩的游戏，可不可以拿到 Linux 去玩？

当然不行！因为游戏也是一个应用程序 (application)，他必须要使用到核心所提供的工具来开发他的游戏， 所以这个游戏是不可在不同的平台间运作的。除非这个游戏已经进行了移植。

## 1.4 Linux 本身仅是一个核心与相关的核心工具而已，不过，他已经可以驱动所有的硬件， 所以，可以算是一个很普通的操作系统了。经过其他应用程序的开发之后，被整合成为 Linux distribitions。请问众多的distributions 之间，有何异同？

**相同**：

1. 同样使用 http://www.kernel.org 所释出的核心； 
2. 支持同样的标准，如 FHS、LSB 等； 
3. 使用几乎相同的自由软件 (例如 GNU 里面的 gcc/glibc/vi/apache/bind/sendmail... )；
4. 几乎相同的操作接口 (例如均使用 bash/KDE/GNOME 等等)。

**不同**：

1. 使用的 kernel 与各软件的版本可能会不同；
2. 各开发商加入的应用工具不同，使用的套件管理模式不同(dpkg 与 RPM)

## 1.5 Unix 是谁写出来的？ GNU 计划是谁发起的？

Unix 是 Ken Thompson 写的，1973 年再由 Dennis Ritchie 以 C 语言改写成功。 

至于 GNU 与 FSF 则是 Richard Stallman 发起的。

## 1.6 GNU 的全名为何？他主要由那个基金会支持？

GNU 是 GNU is Not Unix 的简写，是个无穷循环！ 

另外，这个计划是由自由软件基金会 (Free Software Foundation, FSF) 所支持的！ 两者都是由 Stallman 先生所发起的！

## 1.7 何谓多人 ( Multi-user ) 多任务 ( Multitask )？

Multiuser 指的是 Linux 允许多人同时连上主机之外，每个用户皆有其各人的使用环境，并且可以同时使用系统的资源！

Multitask 指的是多任务环境，在 Linux 系统下， CPU 与其他例如网络资源可以同时进行多项工作， Linux 最大的特色之一即在于其多任务时，资源分配较为平均！

## 1.8 简单说明 GNU General Public License ( GPL ) 与 Open Source 的精神：

1. GPL 的授权之软件，乃为自由软件（Free software），任何人皆可拥有他； 
2. 开发 GPL 的团体(或商业企业)可以经由该软件的服务来取得服务的费用； 
3. 经过 GPL 授权的软件，其属于 Open source 的情况，所以应该公布其原始码； 
4. 任何人皆可修改经由 GPL 授权过的软件，使符合自己的需求； 
5. 经过修改过后 Open source 应该回馈给 Linux 社群。

## 1.9 什么是 POSIX ?为何说 Linux 使用 POSIX 对于发展有很好的影响？

POSIX 是一种标准规范，主要针对在 Unix 操作系统上面跑的程序来进行规范。若你的操作系统符合 POSIX ，则符合 POSIX 的程序就可以在你的操作系统上面运作。 

Linux 由于支持 POSIX ，因此很多 Unix 上的程序可以直接在 Linux 上运作， 因此程序的移植相当简易！也让大家容易转换平台，提升 Linux 的使用率。

## 1.10 简单说明 Linux 成功的因素？

1. 藉由 Minix 操作系统开发的 Unix like ，没有版权的纠纷；
2. 藉助于 GNU 计划所提供的各项工具软件， gcc/bash 等；
3. 藉由 Internet 广为流传；
4. 藉由支持 POSIX 标准，让核心能够适合所有软件的开发；
5. 托瓦兹强调务实，虚拟团队的自然形成！
