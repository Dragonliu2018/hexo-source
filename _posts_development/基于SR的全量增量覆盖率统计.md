---
title: 基于SR的全量/增量覆盖率统计
date: 2021-09-27 09:44:22
tags: 
  - 代码覆盖率
categories:
    - [软件开发]
    - [C/C++]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
toc: true #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
---

# 1 情景导入

2021.09.26华为实习任务复现，要求实现基于SR的全量/增量覆盖率统计。

源代码：https://gitee.com/dragon-liu/RB_01

# 2 部门现状

1. 当前的代码覆盖率只能基于整个代码库进行统计，无法便捷跟踪每个需求引起的覆盖率变化，版本管理成本很高。

   > **方案**：基于SR的全量/增量覆盖率统计

2. 其次，脚本使用Perl语言编写，晦涩难懂，没有维护人员。

   > **编程语言**：Shell + Python

# 3 解决方案

1. 通过git跟踪不同需求引入的代码变更
2. 基于变更代码生成差异文件：强行cherry-pick
   * 无冲突，直接做全量/增量分析，结果是准确的
   * 有冲突需，自己获取差异代码行，结果是近似的
     * 根据SR获取commit id列表
     * 以最后一笔提交为标准
     * 倒序遍历commit id，找出每次提交的代码行（倒序的原因：一般的，越新的commit对当前代码库影响越大）
3. 利用lcov以及addlcov生成对应需求的全量/增量覆盖率报告
4. 通过genhtml工具和python工程库生成html报告
5. 借助Jenkins定时运行脚本与发送邮件

# 4 Jenkins使用

## 4.1 Ubuntu20安装Jenkins

* [如何在 Ubuntu 20.04 上安装 Jenkins](https://cloud.tencent.com/developer/article/1666282)
* [Ubuntu环境下Jenkins安装、卸载、配置](https://blog.csdn.net/qq_35114214/article/details/88747597)

## 4.2 配置构建任务

* [Jenkins配置git仓库](https://blog.csdn.net/qq_42559485/article/details/108511066)
* [Jenkins 之插件 Publish HTML reports 的使用](https://blog.csdn.net/u012599988/article/details/81871596)

## 4.3 邮件发送

* [jenkins构建后邮件发送](https://blog.csdn.net/songjiaping/article/details/51496977)
* [jenkins构建邮件自动发送，测试邮件发送成功，构建项目邮件发送不成功的问题](https://www.cnblogs.com/lelexiong/p/9037807.html)
* [jenkins构建任务后发送邮件](https://www.cnblogs.com/t-ae/p/10186157.html)
* [jenkins发送邮件失败“No emails were triggered”](https://blog.csdn.net/weixin_34199405/article/details/94286501)

本地复现使用的Jenkins自带的邮件提醒功能；华为实习过程中，自己编写脚本进行邮件提醒。

具体实现详见 博客：`Jenkins配置定时发送邮件功能`

# 关于报错

```sh
###强制回滚到某版本
git reset --hard 36409abd22dcc9f9252152b34fa9e929f7c883ec
```

```sh
###解决：https://stackoverflow.com/questions/3411048/unexpected-operator-in-shell-programming/3411061
./srCover.sh: 17: [: h: unexpected operator 
./srCover.sh: 21: [: h: unexpected operator
```
