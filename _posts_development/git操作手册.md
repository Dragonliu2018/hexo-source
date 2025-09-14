---
title: git操作手册
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
date: 2022-04-13 10:17:22
---

# 1 命令

* show-log
* show-diff
* pull：大致上 git pull = git fetch + git merge
* commit
* push
* rebase：可以对某一段线性提交历史进行编辑、删除、复制、粘贴 [git rebase命令](https://www.yiibai.com/git/git_rebase.html)
* delete（rm）
* revert
* 新建本地分支
* 推到远程分支
* 切分支
* cherry-pick
* 解冲突
* .gitignore

# 2 举例

## 2.1 对方更新过代码，然后自己也需要上传新增文件

```sh
# 拉下更新代码
git pull
git pull origin master:master
# 详细的。。。
git pull <远程主机名> <远程分支名>:<本地分支名>
# 新增文件
···
# 将所有新增文件添加到暂存区
git add .
# 将暂存区内容添加到本地仓库中
git commit -m "注释"
# 从将本地的分支版本上传到远程并合并
git push
git push origin master:master
# 详细的。。。
git push <远程主机名> <本地分支名>:<远程分支名>
git push <远程主机名> <本地分支名> # 如果本地分支名与远程分支名相同，则可以省略冒号
```

## 2.2 贡献自己的代码

**commit->pull->push->MR**

**冲突：resolved->修改（edited）->完成（resolved）**

# X 参考

* [菜鸟教程——git](https://www.runoob.com/git/git-tutorial.html)
