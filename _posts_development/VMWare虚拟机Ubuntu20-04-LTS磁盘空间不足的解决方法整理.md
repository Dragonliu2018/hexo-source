---
title: VMWare虚拟机Ubuntu20.04 LTS磁盘空间不足的解决方法整理
tags:
  - Ubuntu
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-02 17:38:02
---

# 1 引入

之前设置磁盘大小为40G，现在搭建cuckoo环境，磁盘空间不足。

在Ubuntu虚拟机的命令行输入 `df -l` 查看磁盘使用情况：

```sh
➜  Desktop df -l
Filesystem     1K-blocks     Used Available Use% Mounted on
udev             4019184        0   4019184   0% /dev
tmpfs             810936     2032    808904   1% /run
/dev/sda5      102431704 46674596  51153504  99% /
```

可以看到 `/dev/sda5` 对应的根目录 `/` 已经满载了。

下面是一些解决方法：

# 2 删除回收站所有文件

使用下面命令即可：

```sh
sudo rm -rf ~/.local/share/Trash/
```

# 3 删除日志文件

删除所有日志文件：（可能存在权限不够，无法删除的情况）

```sh
sudo /dev/null > /var/log/**.log
```

下面这个可以推荐使用：（删除30天之前的旧文件）

```sh
sudo find /var/log/ -type f -mtime +30 -exec rm -f {} \;
```

# 4 删除快照（对磁盘空间大小无影响）

但是对于Ubuntu虚拟机整体目录大小有关。

# 5 删除无关用户

使用下面命令即可：

```sh
sudo userdel -r <newuser>
```

# 6 挂载新磁盘

拓展磁盘空间：[链接](https://blog.csdn.net/weixin_40641735/article/details/88932720)

设置挂载的文件夹访问权限：`sudo chmod 777 ×××` （每个人都有读和写以及执行的权限）

# 7  拓展磁盘空间（不改变原有磁盘内容，推荐）

[跳转](https://blog.csdn.net/weixin_39510813/article/details/78387334?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_default)

# 8 总结

扩展磁盘空间完成，df再看一下：

```sh
➜  Desktop df -l
Filesystem     1K-blocks     Used Available Use% Mounted on
udev             4019184        0   4019184   0% /dev
tmpfs             810936     2032    808904   1% /run
/dev/sda5      102431704 46674596  51153504  48% /
```

现在已经完成任务。
