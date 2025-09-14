---
title: Windows下安装Redis
tags:
categories:
  - [环境与工具]
  - [数据库, Redis]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-15 16:13:23
---

# 1 引入

最近在Windows上运行若依系统，需要配置Redis，但是这台笔记本上还没安装，整理下。

Redis官网：http://www.redis.cn/

主要copy于 [Windows中Redis的下载安装与修改密码并启动](https://blog.csdn.net/BADAO_LIUMANG_QIZHI/article/details/107486313)

# 2 下载安装包

Github下载安装包：[Redis Installer](https://github.com/microsoftarchive/redis/releases)

选择 zip 和 msi 下载都可以，测试使用的 msi。

# 3 安装Redis

1. 双击下载好的安装文件`Redis-x64-3.0.504.msi`打开安装程序

2. 安装时注意将路径加到PATH中：

   <img src="https://s2.loli.net/2022/02/15/DI97iLym3PEAgKQ.png" width = "500" height = "400" alt="图片名称" align=center id=88 />

3. 其他的，如端口等默认选项即可：

   <img src="https://s2.loli.net/2022/02/15/qtfIWCcn57aK9iV.png" width = "500" height = "400" alt="图片名称" align=center id=89 />


# 4 测试

1. **启动服务端**：在上述安装过程中已经自启动Redis了，如果没启动，则在Redis安装目录打开命令行并执行：`redis-server.exe redis.windows.conf`

   <img src="https://s2.loli.net/2022/02/15/26qd8Am9QscVbBh.png" width = "700" height = "300" alt="图片名称" align=center id=97 />

2. **启动客户端**：

   ```sh
   # 这里的123456是上面自己设置的密码
   D:\developer\database\redis\soft>redis-cli.exe -h 127.0.0.1 -p 6379 -a 123456
   127.0.0.1:6379> ping
   PONG
   127.0.0.1:6379>
   ```

# 5 Redis管理软件（Redis Desktop Manager）

1. 从`0.9.3.817 `版本后就开始收费了。当然，源码始终是开源的，需要自行编译，有已经编译好的版本：
   * [新版的RedisDesktopManager下载](https://springboot.io/t/topic/2394)
   * [Github-AnotherRedisDesktopManager](https://github.com/qishibo/AnotherRedisDesktopManager/releases) （实验中使用的软件）
   
2. 新建连接，默认即可：

   <img src="https://s2.loli.net/2022/02/15/41cJfXTdwL75Zzj.png" width = "600" height = "400" alt="图片名称" align=center id=90 />

3. 运行完若依系统后，已经有了数据：

   <img src="https://s2.loli.net/2022/02/15/jwqznTsJZMLPF21.png" width = "600" height = "400" alt="图片名称" align=center id=96 />
