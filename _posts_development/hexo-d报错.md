---
title: hexo d报错
tags:
  - Hexo
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-19 11:10:19
---

# 1 报错

hexo d 命令报错：

```sh
/c/Users/Dragon Liu/.ssh/config: line 5: Bad configuration option: password
/c/Users/Dragon Liu/.ssh/config: terminating, 1 bad configuration options
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
[41mFATAL[49m Something's wrong. Maybe you can find the solution here: [4mhttps://hexo.io/docs/troubleshooting.html[24m
[33mError: Spawn failed
    at ChildProcess.<anonymous> (E:\01 Blog_work\Hexo-Blog\node_modules\hexo-deployer-git\node_modules\hexo-util\lib\spawn.js:51:21)
    at ChildProcess.emit (events.js:200:13)
    at ChildProcess.cp.emit (E:\01 Blog_work\Hexo-Blog\node_modules\hexo-deployer-git\node_modules\cross-spawn\lib\enoent.js:34:29)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:272:12)[39m
```

# 2 解决

## 2.1 情况1

`ssh -T git@github.com` 测试无法连接：

```sh
E:\01 Blog_work\Hexo-Blog>ssh -T git@github.com
C:\\Users\\Dragon Liu/.ssh/config: line 5: Bad configuration option: password
C:\\Users\\Dragon Liu/.ssh/config: terminating, 1 bad configuration options
```

原因是之前vscode下载插件Remote - SSH时配置了此文件：

<img src="https://s2.loli.net/2022/04/19/EF7GobKALXv1jr3.png" width = "800" height = "300" alt="图片名称" align=center id=186 />

删除`config`文件即可测试成功：

```sh
E:\01 Blog_work\Hexo-Blog>ssh -T git@github.com
Warning: Permanently added the ECDSA host key for IP address '140.82.112.3' to the list of known hosts.
Hi Dragonliu2018! You've successfully authenticated, but GitHub does not provide shell access.
```

然后`hexo d`也可以成功。

## 2.2 情况2

hexo原本部署到GitHub，现在需要另一个hexo-blog来写Algorithm部分，然后部署到gitee，hexo d后出现报错。

`ssh -T git@gitee.com` 测试无法连接：

```sh
E:\01 Blog_work\Hexo-Blog-Algorithm>ssh -T git@gitee.com
Warning: Permanently added the ED25519 host key for IP address '180.97.125.228' to the list of known hosts.
git@gitee.com: Permission denied (publickey).
```

原因是没有在 Gitee 中添加 SSH 公钥，复制`id_rsa_pub` 文件内容，然后打开 gitee 个人设置里面的 安全设置 - SSH公钥，标题可以随便取，把粘贴的内容复制到公钥里面，点击确定就可以：

<img src="https://s2.loli.net/2022/04/19/yJIVgfYDMd2A7ne.png" width = "900" height = "250" alt="图片名称" align=center id=187 />

<img src="https://s2.loli.net/2022/04/19/75BPQXYca9IuO2C.png" width = "900" height = "500" alt="图片名称" align=center id=188 />

此时测试成功：

```sh
E:\01 Blog_work\Hexo-Blog-Algorithm>ssh -T git@gitee.com
Hi [36;01mDragon-Liu[0m! You've [32msuccessfully[0m authenticated, but GITEE.COM does not provide shell access.
```

然后`hexo d`即可成功。

> **吐槽**：
>
> 1. 很容易部署失败：提示可能包含违禁违规内容。
> 2. 后期如果新增了文章，执行 `hexo g -d`后 ，还需要更新 Gitee Pages 服务

# X 参考

* [Hexo 部署到 Gitee](https://blog.csdn.net/qq_38157825/article/details/112783631)
