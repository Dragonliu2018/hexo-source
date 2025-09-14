---
title: Pycharm(IntelliJ系列)的Low Memory问题
tags:
categories:
  - [环境与工具]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-11 11:22:48
---

# 1 问题引入

最近在跑恶意样本可视化的脚本，Pycharm出现：

```sh
Low Memory
The IDE is running low on memory and this might affect performance. Please consider increasing available heap.
```

原因是IDE内存不足（之前给pycharm设定了2048M，跑大项目会出现上述问题）。

主要copy于 [idea 的Low Memory问题](https://www.cnblogs.com/baby123/p/15626312.html)

# 2 解决方法

下面的方法适用于IntelliJ系列产品，如Pycharm、idea等。

## 2.1 直接配置

<img src="https://s2.loli.net/2022/02/11/ulXpqTxoQSt1mRB.png" width = "800" height = "200" alt="图片名称" align=center id=68 />

选择上图里的 Configure，修改堆大小：

<img src="https://s2.loli.net/2022/02/11/luJ4ba1mBA3G2hE.png" width = "300" height = "200" alt="图片名称" align=center id=69 />

重启IDE生效。

## 2.2 通过help配置修改

 菜单 Help -> 选择 Edit Custom VM Options，会打开 pycharm64.exe.vmoptions文件，修改配置即可。

<img src="https://s2.loli.net/2022/02/11/EgGpOs8Miow1VSe.png" width = "600" height = "400" alt="图片名称" align=center id=70 />

此时发现`Change Memory Settings`中已经发生了变化：

<img src="https://s2.loli.net/2022/02/11/KPSpc5fVvUZR7ON.png" width = "600" height = "400" alt="图片名称" align=center id=71 />

最后重启IDE生效。
