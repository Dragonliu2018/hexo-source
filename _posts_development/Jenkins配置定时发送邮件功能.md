---
title: Jenkins配置定时发送邮件功能
tags:
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-16 23:48:10
---

# 0 安装Jenkins

[如何在 Ubuntu 20.04 上安装 Jenkins](https://cloud.tencent.com/developer/article/1666282)

Jenkins 是最流行的，开源的，基于 Java 的自动化服务器，它允许你很容易地设置一个[持续集成](https://cloud.tencent.com/product/coding-ci?from=10680)和持续发布的管道。

持续集成 (CI)是一个 DevOps 实践。当团队成员正常提交代码到版本控制仓库时，会运行自动化构建和测试。持续发布（CD）是一系列实践，当代码修改后，自动构建，测试，并且发布到生产环境中。

Jenkins 可以被作为一个独立应用安装，作为 Java servlet 容器（例如 Apache Tomcat）中的一个 servlet 安装，或者 以 Docker 容器的形式运行。

独立服务的形式安装 Jenkins：

# 1 发送邮件

## 1.1 设置参数

为了使得下面脚本成功执行，需要设置以下参数：

* **Receiver** 邮件接收邮箱
* **Copyer** 邮件抄送邮箱
* **Subject** 邮件主题
* **ReportLink** 邮件链接
* **ReportFile** 邮件报告文件（本次测试中未使用）

样例如下：

<img src="https://s2.loli.net/2022/04/17/jkUSXfEdhbH4C6M.png" width = "700" height = "500" alt="图片名称" align=center id=162 />

## 1.2 运行脚本

该脚本在`01_RB`工程中已存在，根据需求修改即可。路径：`test/install/AutoInstallRB/icp/send_report_email.py`

运行脚本：

```sh
python3 send_report_email.py -u "用户ID/codehub密码的Base64编码" "本人邮箱" "$Receiver" "$Copyer" "$ReportFile" "$Subject" "$ReportLink"
```

`send_report_email.py`脚本路径目录：`E:\04 Worker_work\01 实习\202109-华为C++后端实习\hw-addlcov\CBS-SW\01_RB\scripts`

# 2 Jenkins 定时任务时间设置

## 2.1 定时任务的时间格式

定时任务的时间格式设定遵循的是cron的语法规则。具体的格式划分为5个字段：分别为 分钟、小时、日、月、星期、通过tab或者空格进行隔离区分

|             字段              |   *   |  *   |    *    |    *    |   *   |
| :---------------------------: | :---: | :--: | :-----: | :-----: | :---: |
|             含义              | 分钟  | 小时 |  日期   |  月份   | 星期  |
|           取值范围            | 0-59  | 0-23 | 1月30日 | 1月11日 |  0-7  |
|           **实例**            |       |      |         |         |       |
|      每隔15分钟执行一次       | H/15  |  *   |    *    |    *    |   *   |
|       每隔2小时执行一次       |   H   | H/2  |    *    |    *    |   *   |
|        每隔3天执行一次        |   H   |  H   |   H/3   |    *    |   *   |
| 每隔3天执行一次（每月1-15号） |   H   |  H   | 1-15/3  |    *    |   *   |
|      每隔1、3、5执行一次      |   H   |  H   |    *    |    *    | 1,3,5 |
|           **规则**            |       |      |         |         |       |
|         指定时间范围          |  a-b  |      |         |         |       |
|         指定时间间隔          |   /   |      |         |         |       |
|         指定变量取值          | a,b,c |      |         |         |       |

参数含义：

1. *代表 匹配范围内所有值
2. M-N 匹配M~N范围内所有值
3. M-N/X 或者 */X 在指定M~N范围内或整个有效区间内每隔X构建一次
4. A,B,…,Z 匹配多个值

## 2.2 一些常用的时间规则

|                           时间规则                           |      规则内容       |
| :----------------------------------------------------------: | :-----------------: |
|                     每天上午9点定时构建                      |      H 9 * * *      |
|                每天上午11点和下午6点定时构建                 |    H 11,18 * * *    |
|            每个小时的前30分钟内，每10分钟构建一次            | H(0-29)/10 * * * *  |
|        周一到周五，9-16点，每2小时的第45分钟构建一次         |  45 9-16/2 * * 1-5  |
| 每个工作日从早上9点到下午5点这段时间内每间隔2小时之间的某刻。(或许在上午10:38, 下午12:38, 下午2:38 , 下午4:38) | H H(9-16)/2 * * 1-5 |
|          每月（除了12月）从1号到15号这段时间内某刻           |   H H 1,15 1-11 *   |

## 2.3 样例

<img src="https://s2.loli.net/2022/04/17/YfT4HG5mLbhXBPF.png" width = "800" height = "500" alt="图片名称" align=center id=163 />

