---
title: Windows下安装MySQL
tags:
categories:
  - [环境与工具]
  - [数据库, MySQL]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-15 14:02:40
---

# 1 引入

最近在Windows上运行若依系统，需要配置MySQL，但是这台笔记本上还没安装，整理下。

主要copy于 [windows10上安装mysql（详细步骤)](https://blog.csdn.net/zhouzezhou/article/details/52446608)

# 2 下载安装包

官网下载：[MySQL Installer](https://dev.mysql.com/downloads/installer/)

<img src="https://s2.loli.net/2022/02/15/G5fRJ39EiONYtPk.png" width = "500" height = "300" alt="图片名称" align=center id=77 />

下载速度较慢，使用迅雷下载提速。

# 3 安装Mysql

1. 双击下载好的mysql安装文件`mysql-installer-community-8.0.28.0.msi`打开安装程序，打开后需要稍等一下

2. 选择安装类型（根据个人需要）：此处我只需要server，所以选择`Custom`

   <img src="https://s2.loli.net/2022/02/15/7WyjMUGDVnhOAqS.png" width = "500" height = "400" alt="图片名称" align=center id=78 />

3. Select Products：（注意这个地方把软件安装目录和数据目录换成非C盘，因为毕设后期数据量可能比较大）

   <img src="https://s2.loli.net/2022/02/15/HlXf5NxgyJtszQb.png" width = "500" height = "400" alt="图片名称" align=center id=79 />

4. 点击“Execute”（执行）开始安装，安装过程中会显示安装的Progress（进度），等待安装完成后Status会显示Complete，mysql图标前会出现一个绿色的勾，然后点击“Next”按钮进入产品配置界面：

   <img src="https://s2.loli.net/2022/02/15/T2tvdulYaBz7gSh.png" width = "500" height = "400" alt="图片名称" align=center id=80 />

5. `Type and Networking`：默认即可

   <img src="https://s2.loli.net/2022/02/15/wWPQ8dqK2MUaHso.png" width = "500" height = "400" alt="图片名称" align=center id=81 />

6. `Authentication Method`：安装目的是学习使用，所以使用下面的选项即可：

   <img src="https://s2.loli.net/2022/02/15/E9vOgrqcRf1B4b2.png" width = "500" height = "400" alt="图片名称" align=center id=82 />

7. 设置密码等默认操作即可，其中mysql server的名称默认是`MySQL80`，默认开机自启动。

# 4 配置mysql环境变量（非必要）

 说明：给mysql配置环境变量后我们就可以在cmd里运行mysql（开启、停止等操作）

1. 选中系统变量中的`path`，在path值开头处输入mysql安装目录下的bin文件夹所在路径（默认是`C:\Program Files\MySQL\MySQL Server x.x`，但是此处我做了修改），保存退出；

2. 测试是否配置成功：打开cmd，输入

   ```sh
   C:\Users\Dragon Liu>mysql -u root -p
   Enter password: ****
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 10
   Server version: 8.0.28 MySQL Community Server - GPL
   
   Copyright (c) 2000, 2022, Oracle and/or its affiliates.
   
   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.
   
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   
   mysql>
   ```

   测试成功。

   因为上面安装时默认开机自启动，所以这里mysql已经启动了，如果没启动可以管理员运行cmd，然后输入：

   ```sh
   C:\Windows\system32>net start MySQL80
   请求的服务已经启动。
   
   请键入 NET HELPMSG 2182 以获得更多的帮助。
   ```

# 5 MySQL管理软件（Navicat for MySQL）

1. 安装正版Navicat for MySQL：[破解版](https://www.downkuai.com/soft/112947.html)

2. 打开Navicat for MySQL

3. 新建一个连接，填写连接信息：

   <img src="https://s2.loli.net/2022/02/15/yiUoY5KaZOpRWrq.png" width = "400" height = "500" alt="图片名称" align=center id=83 />
