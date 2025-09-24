---
title: '[Tool] github 使用技巧汇总'
categories:
  - [Tool, git]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-24 22:45:16
tags:
---

# 新增 SSH 密钥到 GitHub 帐户
1. 生成 key，一路默认即可：

   ```shell
   ➜  .ssh ssh-keygen
   ```

2. 复制 id_rsa.pub 文件中的内容到 GitHub：

   ```shell
   ➜  .ssh cat id_rsa.pub
   ```

3. 测试是否配置成功：

   ```shell
   ➜  .ssh ssh -T git@github.com
   Hi Dragonliu2018! You've successfully authenticated, but GitHub does not provide shell access.
   ```

   