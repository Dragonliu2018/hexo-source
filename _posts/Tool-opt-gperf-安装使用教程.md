---
title: '[Tool][opt] gperf 安装使用教程'
categories:
  - [Tool,opt]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-26 00:28:23
tags:
---

# 1 介绍

https://github.com/gperftools/gperftools

[gperftools](https://github.com/gperftools/gperftools) 是谷歌推出的一套非常强大的性能分析工具集. 它主要有这三个功能:

- 分析 CPU 性能, 能够统计出一段时间内各个函数的执行时间, 帮助我们找出耗时的代码;
- 分析内存占用情况, 能够统计出某一时刻各个函数分配的内存大小, 帮助我们找出内存占用高的代码, 也能帮助我们定位内存泄露;
- 自动检查内存泄露.

gperftools 还包含一个高性能内存分配器 tcmalloc, 我们可以用它代替 glibc 的 ptmalloc. tcmalloc 自带统计功能, 内存分析和检查内存泄露就靠它.

本文介绍 gperftools 在 Linux 下的一些常见的用法. 如果你需要使用 gperftools 分析 Linux (服务器) 程序, 这篇文章可以当作一个 Quick Start.

# 2 安装

**方法1：**

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null
```

**方法2:**

```bash
brew install gperftools
```

# 3 使用

# X 参考

- [**Install "perf" on Mac**](https://stackoverflow.com/questions/23200704/install-perf-on-mac)
- [**使用 gperftools 分析程序性能**](https://luyuhuang.tech/2022/04/10/gperftools.html)