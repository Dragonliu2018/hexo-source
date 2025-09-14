---
title: 'git报错之error: refs/heads/pa2 does not point to a valid object'
tags:
  - Git
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-10 23:31:38
---

# 1 问题引入

目前项目有两个分支：pa1和pa2。pa1分支没有更新到云端，需要更新；pa2分支出现问题。

**目标**：删除pa2分支，将pa1 push到云端。

# 2 解决

1. 切换到pa1分支，`git checkout pa1`

   <img src="https://s2.loli.net/2022/05/10/FZmLMNhPlxiUCd1.png" width = "550" height = "300" alt="图片名称" align=center id=199 />

2. 提交本地pa2修改的代码，出现如下报错：

   ```sh
   $ git add .
   $ git commit
   fatal: could not parse HEAD
   ```

3. 强制切换pa1分支：

   ```sh
   $ git checkout -f pa1
   ```

4. 将pa1分支push到云端，出现报错：

   <img src="https://s2.loli.net/2022/05/10/iNlGDgz2kVKxjoW.png" width = "650" height = "100" alt="图片名称" align=center id=200 />

5. 执行下面的操作再进行push操作，成功执行

   ```sh
   rm .git/refs/heads/pa2
   ```

# 3 参考

* [How do I delete a local git branch when it can't look up commit object in 'refs/heads'?](https://stackoverflow.com/questions/20694882/how-do-i-delete-a-local-git-branch-when-it-cant-look-up-commit-object-in-refs)
* [解决git报错fatal: could not parse HEAD](https://zhuanlan.zhihu.com/p/426003354)
* [【git操作】强制切换到本地某个分支](https://blog.csdn.net/SMonkeyKing/article/details/89850416)
