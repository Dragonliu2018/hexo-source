---
title: Hexo博客域名
tags:
  - Hexo
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-01 16:19:26
---

# 1 问题引入

今天访问博客失败，原因是之前在[Freenom](https://www.freenom.com/zh/freeandpaiddomains.html)上申请的域名`dragonliu.tk`已经过期。登陆再次申请时，发现已付费：

<img src="https://s2.loli.net/2022/03/01/nCtPu82NslyU4pX.png" width = "700" height = "400" alt="图片名称" align=center id=99 />

# 2 解决

## 2.1 使用github默认的域名

将`source`目录下的`CNAME`删掉，然后重新提交博客即可`hexo g -d`，然后访问https://dragonliu2018.github.io/。

<img src="https://s2.loli.net/2022/03/01/ftXbFJzoDOeYAMy.png" width = "800" height = "400" alt="图片名称" align=center id=100 />

## 2.2 重新申请其他域名

1. 申请到一个新域名`dragonliu.gq`（吐槽：这个网站需要翻墙，访问速度慢，而且老是自动登出，订单容易提交失败，域名到期无提醒。）此时这个域名还Ping不通：

   ```sh
   E:\01 Blog_work\Hexo-Blog>ping dragonliu.gq
   Ping 请求找不到主机 dragonliu.gq。请检查该名称，然后重试。
   ```

2. 在Freenom上配置Domain：

   <img src="https://s2.loli.net/2022/03/01/XbWzqHvLkjuRxFl.png" width = "600" height = "300" alt="图片名称" align=center id=103 />

   <img src="https://s2.loli.net/2022/03/01/k4yW6ULHx7MoTAi.png" width = "600" height = "200" alt="图片名称" align=center id=102 />

   这里填入github博客默认域名：

   <img src="https://s2.loli.net/2022/03/01/juDLbsxv4TKzXlp.png" width = "600" height = "300" alt="图片名称" align=center id=104 />

​	等待几分钟后可以Ping通：

```sh
E:\01 Blog_work\Hexo-Blog>ping dragonliu.gq

正在 Ping dragonliu.gq [185.199.108.153] 具有 32 字节的数据:
来自 185.199.108.153 的回复: 字节=32 时间=58ms TTL=53
来自 185.199.108.153 的回复: 字节=32 时间=74ms TTL=53
来自 185.199.108.153 的回复: 字节=32 时间=74ms TTL=53
来自 185.199.108.153 的回复: 字节=32 时间=58ms TTL=53

185.199.108.153 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 58ms，最长 = 74ms，平均 = 66ms
```

4. 在`source`目录下的`CNAME`文件改为对应新域名：

   ```sh
   dragonliu.gq
   ```

5. 然后重新提交博客即可`hexo g -d`，就可以访问新域名了 `dragonliu.gq`

# X 参考

* [超级详细Hexo+GitHub+阿里云域名的博客搭建教程，新手也能轻松学会](https://segmentfault.com/a/1190000021979631)
