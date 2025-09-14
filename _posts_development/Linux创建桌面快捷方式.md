---
title: Linux创建桌面快捷方式
tags:
categories:
  - 环境与工具
  - [操作系统, Linux]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-15 19:10:16
---

# 1 引入

在windows下创建桌面快捷方式比较容易，右键即可。在Linux下创建快捷方式就比较麻烦了，整理下。

> 这种方法在centos7和redhat系统中测试成功；
>
> 在Ubuntu上失败，有个拙劣的方法：将下面的`go2dir.sh`和`go2url,sh`脚本放到桌面，当作快捷方式。

# 2 跳转指定文件夹

1. 新建shell脚本`go2dir.sh`，具有文件夹跳转功能，内容如下：

   ```sh
   #!/bin/sh
    
   nautilus /home/dragon/cpp
   ```

   nautilus是终端命令，后面是指定要跳转的文件夹路径。

   将脚本添加执行权限：

   ```sh
   chmod +x go2dir.sh
   ```

 2. 在桌面上新建`GotoDir.desktop`文件，内容如下，关注`Exec`和`Icon`变量，代表脚本和图标：

    ```sh
    [Desktop Entry]
    Encoding=UTF-8
    Name=GotoDir
    Comment=Go To My Dir
    Exec=/home/dragon/go2dir.sh
    Icon=/home/dragon/go2dir.png
    Terminal=false
    StartupNotify=true
    Type=Application
    Categories=Application;Development;
    ```

	2. 在桌面上双击快捷方式，即可实现一键跳转指定文件夹。

# 3 跳转指定网页(url)

1. 新建shell脚本`go2url.sh`，具有网页跳转功能（火狐浏览器），内容如下：

   ```sh
   #!/bin/sh
    
   firefox --new-window www.baidu.com
   ```

   nautilus是终端命令，后面是指定要跳转的文件夹路径。

   将脚本添加执行权限：

   ```sh
   chmod +x go2url.sh
   ```

 2. 在桌面上新建`GotoUrl.desktop`文件，内容如下，关注`Exec`和`Icon`变量，代表脚本和图标：

    ```sh
    [Desktop Entry]
    Encoding=UTF-8
    Name=GotoUrl
    Comment=Go To My Url
    Exec=/home/dragon/go2url.sh
    Icon=/home/dragon/go2url.png
    Terminal=false
    StartupNotify=true
    Type=Application
    Categories=Application;Development;
    ```

 3. 在桌面上双击快捷方式，即可实现一键跳转指定网页。

# X 参考

* [CentOS 7 新建桌面快捷方式，实现一键跳转到指定的文件夹路径](https://blog.csdn.net/libaineu2004/article/details/83757377)
* [从命令行 Linux 启动 Firefox](https://zditect.com/article/2115540.html)
