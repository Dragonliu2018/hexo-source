---
title: 借助gitbook与Gitee Pages服务搭建静态网站
tags:
categories:
  - [环境与工具]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-25 17:03:12
---

# 1 引入

在做PA助教，需要将讲义重新部署下，静态网站。讲义仓库在Gitee，所以使用 `gitbook + Gitee Pages` 实现该需求。本文将以自己的`Algorithm`仓库进行演示：

# 2 实现

主要参考：[使用 Gitbook 打造你的电子书](https://zhuanlan.zhihu.com/p/34946169)

## 2.1 本地安装gitbook

### 2.1.1 安装gitbook

环境要求：

- NodeJS（推荐使用v4.0.0及以上版本）
- Windows，Linux，Unix 或 Mac OS X

Windows下在cmd中输入：

```sh
npm install gitbook-cli -g
```

执行下面的命令，查看 GitBook 版本，以验证安装成功：

```sh
gitbook -V
```

### 2.1.2 GitBook 项目结构

GitBook使用简单的目录结构。在 SUMMARY （即 SUMMARY.md 文件）中列出的所有 Markdown / Asciidoc 文件将被转换为 HTML。

一个基本的 GitBook 电子书结构通常如下：

```
.
├── book.json
├── README.md
├── SUMMARY.md
├── chapter-1/
|   ├── README.md
|   └── something.md
└── chapter-2/
    ├── README.md
    └── something.md
```

GitBook 特殊文件的功能：

|    文件     |                 描述                 |
| :---------: | :----------------------------------: |
|  book.json  |     配置数据、包括插件配置(可选)     |
|  README.md  | 电子书的前言或每个文件夹的简介(必选) |
| SUMMARY.md  |           电子书目录(可选)           |
| GLOSSARY.md |       词汇/注释术语列表(可选)        |

### 2.1.3 创建book.json与SUMMARY.md

* 参考：[Gitbook根据目录生成Summary.md](https://www.jianshu.com/p/2160f1ba68a0)

`book.json` 文件需要自己创建，`SUMMARY.md`文件可以使用插件(gitbook-plugin-summary)自动生成。

安装插件：

```sh
npm i gitbook-plugin-summary --save
```

在`Algorithm`项目根目录创建 `book.json` 文件，并输入：

```json
{
    "plugins": [ 
		"summary"
	]
}
```

### 2.1.4 目录顺序

关于数字序号的顺序：`Ch10`、`Ch11`、`Ch1 `、`Ch2`、`Ch3`、`Ch4`...

所以需要修改序号：`Ch1`、`Ch2`、`Ch3 `、`Ch4`、...、`Ch10`、`Ch11`

### 2.1.5 数学公式

使用`mathjax`，但是不支持行内公式，效果不好，后续再看怎么解决这一块。

参考：

* [gitbook使用mathjax](https://zhuanlan.zhihu.com/p/132638228)
* [gitbook采坑笔记（一：Mathjax）](https://blog.csdn.net/qq_42898299/article/details/106779945)

### 2.1.6 构建与启动服务

使用下面的命令，会在项目的目录下生成一个 `_book` 目录，里面的内容为静态站点的资源文件：

```sh
gitbook build
```

使用下列命令会运行一个 web 服务, 通过 `http://localhost:4000/` 可以预览书籍：

```sh
gitbook serve
```

至此，`Algorithm`仓库对应的静态网站已经部署成功。

## 2.2 实现在线访问

在Gitee上打开`Algorithm`仓库，点击`服务`标签，选择`Gitee Pages`图标。

**注意**：Gitee Pages服务需要实名认证，审核需要数个工作日。

<img src="https://s2.loli.net/2022/03/25/w3mcMnqHIf4dyUG.png" width = "800" height = "400" alt="图片名称" align=center id=116 />

 **部署目录需要是`_book`**

现在访问 http://dragon-liu.gitee.io/algorithm 即可。
