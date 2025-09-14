---
title: Linux查看系统信息
tags:
categories:
  - 环境与工具
  - [操作系统, Linux]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-11 00:43:44
---

# 1 系统版本

```sh
(base) [ccyin@localhost ~]$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```

[Linux怎么查看操作系统版本号](https://jingyan.baidu.com/article/425e69e6ada3edff14fc167c.html)

# 2 显卡

```sh
$ nvidia-smi
```

[Linux查看GPU信息和使用情况](https://blog.csdn.net/dcrmg/article/details/78146797)

# 3 CPU

```sh
$ cat  /proc/cpuinfo
```

[linux查看cpu型号的方法](https://www.yisu.com/zixun/132704.html)

# 4 内存

```sh
(base) [ccyin@localhost ~]$ cat /proc/meminfo
MemTotal:       263591864 kB
MemFree:        191960176 kB
MemAvailable:   246338560 kB
Buffers:          119332 kB
Cached:         53866512 kB
...

(base) [ccyin@localhost ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:         257413       15579      187461          27       54372      240565
Swap:          4095           0        4095
```

[linux 查看CPU、内存大小](https://www.programminghunter.com/article/9454887470/)

# 5 硬盘

```sh
(base) [ccyin@localhost ~]$ df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   50G   16G   35G   31% /
devtmpfs                 126G     0  126G    0% /dev
tmpfs                    126G  8.0K  126G    1% /dev/shm
tmpfs                    126G   12M  126G    1% /run
tmpfs                    126G     0  126G    0% /sys/fs/cgroup
/dev/sda2               1014M  182M  833M   18% /boot
/dev/mapper/centos-home  2.2T  1.5T  724G   67% /home
tmpfs                     26G   12K   26G    1% /run/user/42
tmpfs                     26G     0   26G    0% /run/user/1005
tmpfs                     26G     0   26G    0% /run/user/1003
tmpfs                     26G     0   26G    0% /run/user/1002
tmpfs                     26G     0   26G    0% /run/user/1010
/dev/mapper/data1-lvol0  8.0T  4.1G  7.6T    1% /data1
```

[linux 查看CPU、内存大小](https://www.programminghunter.com/article/9454887470/)
