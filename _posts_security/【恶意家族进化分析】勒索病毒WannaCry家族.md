---
title: 【恶意家族进化分析】勒索病毒WannaCry家族
tags:
categories:
  - 恶意代码
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-10 15:48:19
---

# 1 简介

WannaCry（又叫Wanna Decryptor），一种**加密型勒索软件兼蠕虫病毒**（Encrypting Ransomware Worm），大小3.3MB，由不法分子利用NSA（National Security Agency，美国国家安全局）泄露的危险漏洞“EternalBlue”（永恒之蓝）进行传播，为WanaCrypt0r 1.0的变种。

<img src="https://s2.loli.net/2022/03/10/8HeVcJj9ELopBkn.png" width = "600" height = "400" alt="图片名称" align=center id=106 />

恶意软件数据集中的家族标签：`Trojan.Ransom.WannaCryptor.A`、`Trojan.Ransom.WannaCryptor.H`

* [百度百科]()
* [维基百科](https://zh.wikipedia.org/wiki/WannaCry#cite_note-trendmicro-12)

# 2 进化

## 2.1 WanaCrypt0r 1.0



## 2.2 WannaCry



## 2.3 WannaSister



# 3 进化图示

<img src="https://s2.loli.net/2022/03/10/takqNpIlwMs82VT.png" width = "800" height = "500" alt="图片名称" align=center id=107 />

|                        对比项                        |      WannaCry1.0版      |   WannaCry2.0版    |
| :--------------------------------------------------: | :---------------------: | :----------------: |
|                        时间戳                        | 未修改（最早2017.3.27） |       被修改       |
|                         标题                         |   Wanna Decryptor 1.0   | Wana Decrypt0r 2.0 |
| c.wry(配置文件其中包含比特币钱包地址和下载TOR地址等) |          存在           |        存在        |
|          b.wry(!WannaCryptor!.bmp桌面图片)           |          存在           |        存在        |
|           r.wry(!Please Read Me!.txt FAQ)            |          存在           |        存在        |
|              f.wry(测试解压文件的路径)               |          存在           |        存在        |
|             t.wry加密模块是一个DLL文件)              |          存在           |        存在        |
|        u.wry(!WannaDecryptor!.exe可执行程序)         |          存在           |        存在        |
|             m.wry(语言文件是RTF文件格式)             |     存在（1种语言）     |  存在（28种语言）  |
|            s.wnry(释放内嵌TOR包TaskData)             |         不存在          |        存在        |
|             m.vbs(脚本文件创建快捷方式)              |         不存在          |        存在        |
|                     00000000.res                     |          存在           |        存在        |
|               00000000.pky为RSA子公钥                |          存在           |        存在        |
|   00000000.eky是RSA子私钥使用RSA主公钥加密后的文件   |          存在           |        存在        |
|          taskdl.exe删除移动到TMP目录的文件           |         不存在          |        存在        |
|         taskse.exe使远程会话可以看到勒索窗体         |         不存在          |        存在        |
|                     资源解压密码                     |   wcry@123、wcry@2016   |     WNcry@2ol7     |



# X 参考

* https://www.sohu.com/a/140989865_114837
* https://www.qq.com/a/20170614/057733.htm
* https://www.chinanews.com.cn/m/sh/2017/05-16/8225594.shtml
* https://www.antiy.com/response/wannacry.html
* 
