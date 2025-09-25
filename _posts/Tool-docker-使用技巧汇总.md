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

# 导出导入镜像

```shell
# 导出镜像
docker save -o doris.tar apache/doris:build-env-ldb-toolchain-latest

# 导入镜像
docker load -i doris.tar
```