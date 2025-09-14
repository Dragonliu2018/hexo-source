---
title: 《鸟哥的Linux私房菜基础学习篇（第四版）》第3章笔记与习题
tags:
categories:
  - 阅读
  - [操作系统, Linux]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-25 15:56:57
---

> 第3章：安装 CentOS 7.x

# 1 笔记

`见博客 VMware安装CentOS 7虚拟机(待补充)`

[VMware安装Centos7系统（中文图形化模式）](https://blog.csdn.net/renfeigui0/article/details/102497359)

***

* root 的密码等设置都会被记录到 /root/anaconda-ks.cfg，你未来想要重建一个一模一样的系统时，就可以参考该文件来制作。你也可以 google 一下，找 kickstart 这个关键词。 
* CentOS 7 默认使用 xfs 作为文件系统

# 2 习题

## 1.1 Linux 的目录配置以『树状目录』来配置，至于磁盘分区槽(partition)则需要与树状目录相配合！ 请问，在预设的情况下，在安装的时候系统会要求你一定要分区出来的两个 Partition 为何？

就是根目录『/』与 内存交换空间『Swap』 

## 1.2 预设使用 MBR 分区方式的情况下，在第二颗 SATA 磁盘中，分区『六个有用』的分区 (具有 filesystem 的) ，此外，已知有两个 primary 的分区类型！请问六个分区的文件名？

/dev/sdb1(primary)

/dev/sdb2(primary)

/dev/sdb3(extended)

/dev/sdb5(logical 底下皆为 logical)

/dev/sdb6

/dev/sdb7

/dev/sdb8

请注意，5-8 这四个 logical 容量相加的总和为 /dev/sdb3！ 

## 1.3 什么是 GMT 时间？北京时间差几个钟头？

GMT 时间指的是格林尼治时间，称为标准的时间，而北京时间较 GMT 快了 8 小时！

## 1.4 软件磁盘阵列的装置文件名为何？

RAID : /dev/md[0-127];

## 1.5 如果我的磁盘分区时使用 MBR 方式，且设定了四个 Primary 分区，但是磁盘还有空间，请问我还能不能使用这些空间？

不行！因为最多只有四个 Primary 的磁盘分区槽，没有多的可以进行分区了！且由于没有 Extended ，所以自然不能再使用 Logical 分区。
