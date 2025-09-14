---
title: Linux无法识别移动硬盘
tags:
categories:
  - 环境与工具
  - [操作系统, Linux]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-15 23:34:58
---

# 1 引入

Centos7 插入 ntfs 类型的移动硬盘/U盘时，无法访问，提示如下：

```sh
Error mounting xxx at xxx: Filesystem type ntfs not configured in kernel
```

> **注意**：redhat系统使用yum包管理工具，因此下面的解决方法也适用。

# 2 解决方法1

1. 由于 centos 默认没有ntfs的源，因此需要先加源：

   ```sh
   sudo wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
   ```

2. 更新源列表到本地：

   ```sh
   sudo yum update
   ```

3. 进行安装：

   ```sh
   sudo yum install ntfs-3g
   ```

   安装之后无需重启就可以正常使用移动硬盘/U盘了。

# 3 解决方法2

如果只是复制文件到主机，那么只需要cp指令即可：

```sh
cp <源文件地址> <目标地址>
```

> 该方法是从软件杯初审老师哪里学到的。

# X 参考

* [centos7 filesystem type ntfs not configured in kernel](https://blog.csdn.net/chy555chy/article/details/113747253)
