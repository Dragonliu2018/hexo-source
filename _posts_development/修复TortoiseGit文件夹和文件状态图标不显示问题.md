---
title: 修复TortoiseGit文件夹和文件状态图标不显示问题
tags:
  - Git
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-25 18:58:15
---

# 1 引入

今天打开git仓库项目，突然发现文件夹和文件状态图标不再显示。

# 2 解决(修改注册表Overlay的值)

参考：[【随记】修复TortoiseGit文件夹和文件状态图标不显示问题](https://www.cnblogs.com/xiesong/p/5761352.html)

1. 按`Win+R`键打开运行对话框，输入 `regedit` ，打开注册表；

2. 在注册表中找到 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers`

3. 修改tortoise相关的名称（如在名称前加一个空格前缀）使其置于开始位置：

   <img src="https://s2.loli.net/2022/03/25/fvyk7c52IzhD1HQ.png" width = "900" height = "400" alt="图片名称" align=center id=117 />

4. 重启电脑，图标正常显示：

   <img src="https://s2.loli.net/2022/03/25/Ca8BDKsJQLfoqnY.png" width = "900" height = "300" alt="图片名称" align=center id=118 />

# X 参考

* [TortoiseGit 显示文件状态图标](https://blog.csdn.net/mcsbary/article/details/90665192)
* [解决TortoiseGit文件夹和文件状态图标不显示问题](https://blog.csdn.net/quantum7/article/details/84639445)

