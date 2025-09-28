---
title: '[Tool] docker 使用技巧汇总'
date: 2025-09-25 19:56:32
categories:
  - [Tool,docker]
toc: true
mathjax: true
top: false
comments: true
copyright: true
tags:
---

# 安装
## mac
ref: https://www.runoob.com/docker/macos-docker-install.html

【方法1】使用 Homebrew 安装

```shell
brew install --cask --appdir=/Applications docker
```

***

【方法2】手动下载安装

* https://docs.docker.com/docker-for-mac/install/

# 镜像加速

* https://www.runoob.com/docker/macos-docker-install.html

# 导出导入镜像

```shell
# 导出镜像
docker save -o doris.tar apache/doris:build-env-ldb-toolchain-latest

# 导入镜像
docker load -i doris.tar
```