---
title: 物理机安装Ubuntu+Win10双系统
date: 2021-10-27 09:44:22
tags: 
  - Ubuntu
categories:
    - [环境与工具]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/ddddd.jpg
toc: true #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
---

# 0 情景导入

1. 宿舍中闲置一台华硕电脑，为充分利用资源，装上Ubuntu系统(20.04)；
2. 搭建开发环境，练习远程开发；
3. 目前在局域网中即可满足需求，后续可能配置内网穿透。

# 1 制作启动U盘

## 1.1 准备工具

* 4G以上的U盘
* 启动U盘制作工具：Ubuntu官方推荐的[Rufus](https://rufus.ie/zh/)
* Ubuntu系统镜像：[官网](https://ubuntu.com/)；（清华源下载速度快）

## 1.2 制作U盘

运行Rufus，一般情况下，选择插入的U盘`设备`，和刚下载的Ubuntu`镜像`，其它保持默认即可开始U盘制作，等待制作完成即可。

<img src="https://i.loli.net/2021/10/16/X7ljH8y4WbkhcUG.png" width = "500" height = "600" alt="图片名称" align=center id=20 />

# 2 磁盘分区

1. 打开`控制面板`，找到`磁盘管理`；
2. 如果整个卷都用于磁盘分区，则`删除卷`，否则需要`压缩卷`，出现下面的`未分配`即可使用：

> * 删除卷的话并不是抹除这部分空间，只是这部分空间会被系统保留出来，没有任何文件格式，对于这部分空间系统不会进行任何操作。它相当于一个无人区。
> * 压缩卷就是把一个盘里面的空间抽出一部分变成一个没有任何文件格式和名称的盘，你接下来要对压缩出来的盘进行格式化操作，这样压缩出来的卷才可以用。

<img src="https://i.loli.net/2021/10/16/Y21WZODa7ynNouL.png" width = "900" height = "300" alt="图片名称" align=center id=21 />

# 3 安装Ubuntu

## 3.1 U盘启动

1. 将制作好的启动U盘插入到闲置电脑，重启(开机)按`F2`键进入BIOS界面；（进入BIOS需要按的键每个品牌的电脑不同）
2. 在`Boot Priority`栏，将`UEFI:***`选项拖到开始，使其开机启动进入U盘；
3. 保存修改退出即可，一会儿自动重启。

## 3.2 安装系统

1. 从U盘启动后，在启动引导页选择`安装Ubuntu`进入Ubuntu；

2. 一路默认选项，直到`Installation type`选择`Something else`，进入分区配置页面，找到`free space`，也就是在Windows系统中整好的磁盘分区；

   |      分区      | 分区类型 | 分区位置 |     用于     | 挂载点 |      大小      |
   | :------------: | :------: | :------: | :----------: | :----: | :------------: |
   | boot(启动分区) |  主分区  |   起始   | ext4文件系统 | /boot  |     1G-2G      |
   | swap(交换空间) | 逻辑分区 |   起始   |   交换分区   |   无   | 与物理内存一致 |
   |       /        |  主分区  |   起始   | ext4文件系统 |   /    |   剩余的空间   |

   > 只有`/`是必须分区的，其它如果没有必要都不建议建议独立分区，按默认挂在root下就行。

3. 点击下一步完成后续安装即可正确安装Ubuntu。
4. 拔出U盘，重启即可使用双系统。

# 4 配置Ubuntu

## 4.1 apt换源并安装必要软件

1. 点击`Settings` -> `About` -> `Software Updates`，选择合适的镜像；

2. 安装必要软件：

   ```sh
   sudo apt install vim
   sudo apt install net-tools
   sudo apt install make
   sudo apt install git
   ```

## * 4.2 科学上网



## 4.3 配置oh-my-zsh

参考`Win10安装Ubuntu子系统`

## 4.4 xshell连接

1. 使用xshell连接，被拒绝`(port 22): Connection failed`

   ```sh
   # 查看服务器22端口是否启动
   lsof -i:22
   # 若没启动
   sudo apt-get install openssh-server openssh-client 
   # 开启ssh
   service ssh start
   ssh localhost
   # 此时检查22是否启动
   lsof -i:22
   ```

2. 笔记本设置合盖不休眠：修改Login Manager配置文件。

   ```sh
   sudo vim /etc/systemd/logind.conf
   ### 修改 ###
   # 找到其中一行：
   #HandleLidSwitch=suspend
   # 将其改为：
   HandleLidSwitch=ignore
   
   # 重启服务：重启计算机即可
   sudo restart systemd-logind  
   # 或者
   service systemd-logind restart
   # 或者
   sudo shutdown -r now
   ```

# 4.5 查看硬件配置

```sh
# 查看系统内核
uname -a
# 查看CPU
cat /proc/cpuinfo | grep model\ name
# 查看内存
cat /proc/meminfo | grep MemTotal
# 查看显卡
lspci | grep 'VGA'
# 查看声卡
lspci | grep -i 'Audio'
# 查看网卡
lspci | grep -i 'Network'
# 查看硬盘
df -lh
```

# 5 参考

* [装了5次Ubuntu，告诉你win10+Ubuntu双系统的正确打开方式](https://zhuanlan.zhihu.com/p/101307629)
* 

