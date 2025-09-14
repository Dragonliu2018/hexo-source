---
title: git clone失败整理
tags:
  - Git
  - 待补充
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-14 19:15:00
---

# 1 errno 10054

翻墙了，但是仍然出现下面的报错：

```sh
Cloning into 'redash'...
fatal: unable to access 'https://github.com/getredash/redash.git/': OpenSSL SSL_read: Connection was reset, errno 10054
```

可能是因为代码库太大，调高 `git buffer size` 即可：

```sh
# 方法1 全局修改
git config --global http.postBuffer 524288000

# 方法2 只针对当前仓库
git init
git config http.postBuffer 524288000
git remote add origin <REPO URL>
git pull origin master
```

重开cmd，倘若还不成功，重复几次即可。

* [stackoverflow问答](https://stackoverflow.com/questions/46232906/git-clone-error-rpc-failed-curl-56-openssl-ssl-read-ssl-error-syscall-errno)

# 2 443：Timed out

```sh
Cloning into 'redash'...
fatal: unable to access 'https://github.com/getredash/redash.git/': Failed to connect to github.com port 443: Timed out
```

翻墙是一种方法，其他待整理

# 3 OpenSSL SSL_connect: SSL_ERROR_SYSCALL

```sh
Cloning into 'addlcov'...
fatal: unable to access 'https://github.com/Dragonliu2018/addlcov.git/': OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
```

又试了一次，成功下载。
