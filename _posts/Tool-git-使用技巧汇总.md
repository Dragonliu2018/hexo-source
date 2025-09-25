---
title: '[Tool] git 使用技巧汇总'
categories:
  - [Tool, git]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-24 22:21:50
tags:
---

# 将当前修改压到 HEAD commit

```shell
git commit --amend
```

# 取消某个文件的更改

```shell
git checkout -- <file_name>
```

> 参考：https://blog.csdn.net/qq_32907195/article/details/115333898

# 删除某个文件(夹)

```shell
# 推荐
git rm <file_name>

# 相当于
rm <file_name>
git add <file_name>
```

# ignore 已经 push 到远端的文件
【**问题**】

.vscode 文件夹已经 push 到远程，如何 ignore？

***

【**解决**】

1. 从 Git 索引中移除 .vscode 目录（不会删除本地文件夹，只是让 git 不再跟踪它）：
   
    ```bash
    ➜  SG-DQA git:(main) git rm -r --cached .vscode
    rm '.vscode/launch.json'
    rm '.vscode/settings.json'
    ```
2. 添加 `.gitignore` 文件
   
    ```bash
    .vscode/
    ```
3. git add & git commit & git push

# 合并多个 commit

![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-24_23-13-21.png)

```shell
# method-1
git rebase -i origin/main
# method-2
git rebase -i ce75153a87
# method-3: 与要合并的commit数量一致
git rebase -i HEAD~8
```

# patch

```shell
git diff > xx.patch
git apply x.patch
```

# 本地分支 & 远程分支

```shell
# 查看本地分支关联（跟踪）的远程分支之间的对应关系
git branch -vv

# 推送当前分支到远程（没有对应分支）
git push -f --set-upstream origin dragonliu/dictionary_mysql_opt

# 设置本地分支与远程分支的关联
git branch --set-upstream-to=origin/<远程分支名>

```

# 删除分支

```shell
# 删除本地分支
git branch -D fix/authentication

# 删除远程分支
 git push origin --delete fix/authentication
```

# 撤销 commit

```shell
# 撤销本地 commit
git reset --soft HEAD^

# 撤销远程 commit
git reset HEAD^
git push -f
```
# 修改远程仓库链接

```shell
git remote set-url origin git@github.com:Dragonliu2018/doris.git
```


