---
title: 【开源代码学习--若依】Windows下运行若依
tags:
  - 若依
categories:
  - [Java, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-15 11:30:43
---

# 1 引入

最近在为毕设选取后台管理框架，开始打算用自己熟悉的Flask，或者Django，毕竟后边恶意样本的检测和可视化编程主语言会是python，这样一来使用python系的框架会做到无缝衔接。

* **Flask**
  * **redash**：[redash-Github](https://github.com/getredash/redash)、[redash-Gitee](https://gitee.com/mirrors/redash)、[redash-介绍](https://cloud.tencent.com/developer/article/1669226)、[官网](https://redash.io/product/)
  * **Flask-Admin-Dashboard**：[Github](https://github.com/jonalxh/Flask-Admin-Dashboard)
* **Django+Vue**
  * **Medusa**：[Github](https://github.com/Ascotbe/Medusa)、[文档](http://medusa.ascotbe.com/Documentation/#/Installation)
  * [Vue+Django整合](https://www.bilibili.com/video/BV1eK411p7Na/?spm_id_from=333.788.recommend_more_video.4)

但是考虑到这些框架网上参考资料、视频教程较少，上手时间可能较长。所以最后决定使用若依前后端分离版本。

* **简介**：基于SpringBoot、Spring Security、Jwt、Vue的前后端分离的后台管理系统，还有前后端不分离版本、cloud版本
* **官网**：[官网](http://ruoyi.vip/)、[源码](https://gitee.com/y_project/RuoYi-Vue)、[文档](http://doc.ruoyi.vip/ruoyi-vue/)
* **环境要求**：
  * JDK 1.8+ (推荐1.8版本)
  * Mysql (业务数据库) 5.7+ (推荐5.7版本)
  * Redis (缓存数据库) 3.0+
  * Maven 3.0+
  * Node 12+
* **开发工具**：
  * **IDEA**：Spring Boot、Vue
  * **Navicat**：mysql
  * **Another Redis Manager Desktop**：Redis

主要参考教程：[B站——楠哥教你学java](https://www.bilibili.com/video/BV1HT4y1d7oA?from=search&seid=872568368355498822&spm_id_from=333.337.0.0)

> 使用开源项目的步骤：
>
> 1. 下载并运行
> 2. 看懂业务流程
> 3. 进行二次开发

# 2 查看自己的环境

打开`cmd`查看自己的环境，不满足上述 **环境要求** 的做出整改：

```sh
# 查看java版本
E:\>java -version
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)

# 查看maven版本
E:\>mvn -v
'mvn' 不是内部或外部命令，也不是可运行的程序
或批处理文件。

# 查看Node版本
E:\>node -v
v12.5.0

# Redis版本
D:\developer\database\redis\soft>redis-server.exe -v
Redis server v=3.0.504 sha=00000000:0 malloc=jemalloc-3.6.0 bits=64 build=a4f7a6e86f2d60b3

# Mysql版本查看
E:\>mysql --version
mysql  Ver 8.0.28 for Win64 on x86_64 (MySQL Community Server - GPL)
```

# 3 下载并打开

IDEA从git链接导入项目，IDEA 打开总项目Spring Boot——`RuoYi-Vue`和前端项目Vue——`ruo-ui`。

下载源码之后`ruoyi-ui`是前端项目模块，`ruoyi-admin`是后端主模块，其他的是后端用到的其他模块。

`sql`目录下会有两个sql文件，这两个sql文件就是数据文件，需要导入Mysql数据库。

# 4 配置Mysql

1. 安装MySQL：参考博客 [Windows下安装MySQL](https://dragonliu.tk/2022/02/15/Windows%E4%B8%8B%E5%AE%89%E8%A3%85MySQL/)

2. 打开Navicat，新建数据库：（<font color=red>切记在新建数据库时要将字符集设置为utf8不然在执行向数据库插入一些数据含有中文时会报错</font>）

   <img src="https://s2.loli.net/2022/02/15/NoME25r7z9hRUgj.png" width = "600" height = "400" alt="图片名称" align=center id=84 />

3. 新建数据库完成后在数据库上右击-运行sql文件，这里使用的是Navicat连接工具。然后选中上面的sql目录下的`ry_20210908.sql`，点击开始，然后再执行sql目录下的`quartz.sql`；

   <img src="https://s2.loli.net/2022/02/15/htzbPfWQD9wXCV7.png" width = "600" height = "400" alt="图片名称" align=center id=85 />

4. 初始化数据库完成之后，会在数据库下生成一些表，并在表中生成一些数据：

   <img src="https://s2.loli.net/2022/02/15/oSrLQtNIqgvYz2Z.png" width = "1000" height = "400" alt="图片名称" align=center id=86 />

5. 在工程中配置数据库：

   <img src="https://s2.loli.net/2022/02/15/CiJu9dHyLBZoeTf.png" width = "1000" height = "200" alt="图片名称" align=center id=87 />

# 5 配置Redis

1. 安装Redis：参考博客 [Windows下安装Redis](https://dragonliu.tk/2022/02/15/Windows%E4%B8%8B%E5%AE%89%E8%A3%85Redis/)

2. 在工程中配置数据库，若没修改Redis默认配置，无需改动下面的代码：

   <img src="https://s2.loli.net/2022/02/15/d14ySpXGZjfEAO3.png" width = "1000" height = "200" alt="图片名称" align=center id=91 />

# 6 运行后端

找到启动程序，右键运行即可：

<img src="https://s2.loli.net/2022/02/15/gOrfEjoFSdXpNH5.png" width = "1000" height = "200" alt="图片名称" align=center id=92 />

等待一段时间，Console中输出：

```sh
...
16:35:40.364 [restartedMain] INFO  o.a.c.h.Http11NioProtocol - [log,173] - Starting ProtocolHandler ["http-nio-8080"]
16:35:40.534 [restartedMain] INFO  c.r.RuoYiApplication - [logStarted,61] - Started RuoYiApplication in 6.585 seconds (JVM running for 7.614)
(♥◠‿◠)ﾉﾞ  若依启动成功   ლ(´ڡ`ლ)ﾞ  
 .-------.       ____     __        
 |  _ _   \      \   \   /  /    
 | ( ' )  |       \  _. /  '       
 |(_ o _) /        _( )_ .'         
 | (_,_).' __  ___(_ o _)'          
 |  |\ \  |  ||   |(_,_)'         
 |  | \ `'   /|   `-'  /           
 |  |  \    /  \      /           
 ''-'   `'-'    `-..-'              
16:35:41.546 [Quartz Scheduler [RuoyiScheduler]] INFO  o.q.c.QuartzScheduler - [start,547] - Scheduler RuoyiScheduler_$_LAPTOP-NE344QFT1644914139051 started.
16:36:15.811 [http-nio-8080-exec-1] INFO  o.a.c.c.C.[.[.[/] - [log,173] - Initializing Spring DispatcherServlet 'dispatcherServlet'
```

打开浏览器，输入`http://localhost:8080/`：

```sh
欢迎使用RuoYi后台管理框架，当前版本：v3.8.1，请通过前端地址访问。
```

至此，后端运行成功。

# 7 运行前端

在IDEA中打开 `Terminal` 标签，并输入 `npm install --registry=https://registry.npm.taobao.org` 安装依赖：

<img src="https://s2.loli.net/2022/02/15/qAkZxnyW1PbSr7N.png" width = "1000" height = "200" alt="图片名称" align=center id=93 />

等待依赖安装完成，启动服务：

```sh
E:\05 Code\RMDVS\RuoYi-Vue\ruoyi-ui>npm run dev

> ruoyi@3.8.1 dev E:\05 Code\RMDVS\RuoYi-Vue\ruoyi-ui
> vue-cli-service serve

 INFO  Starting development server...
98% after emitting CopyPlugin

 DONE  Compiled successfully in 20312ms                                                                                                                                                               18:05:56


  App running at:
  - Local:   http://localhost:80/
  - Network: http://192.168.1.7:80/

  Note that the development build is not optimized.
  To create a production build, run npm run build.
```

启动成功后，会自动跳转到浏览器 `http://localhost:80/`，然后能出现验证码则是请求后台成功。

<img src="https://s2.loli.net/2022/02/15/4TGiZCcwX827Eno.png" width = "600" height = "400" alt="图片名称" align=center id=94 />

登陆成功，至此前端启动成功。

<img src="https://s2.loli.net/2022/02/15/gdFxAsfShMykPvw.png" width = "800" height = "400" alt="图片名称" align=center id=95 />

至此，若依系统前后端全部启动成功！！

# X 参考

* [若依前后端分离版手把手教你本地搭建环境并运行项目](https://blog.csdn.net/BADAO_LIUMANG_QIZHI/article/details/108465662)
* 
