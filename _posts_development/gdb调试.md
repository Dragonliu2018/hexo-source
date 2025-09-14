---
title: gdb调试
tags:
  - gdb
categories:
  - [环境与工具]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-27 13:24:18
---

# 1 前言

gdb调试是必须要掌握的基本技能。

# 2 安装gdb-peda

gdb-peda是gdb的插件，加强gdb调试能力。

安装方法如下：

```sh
gdb-peda$ set disassembly-flavor att$ git clone htgdb-peda$ set disassembly-flavor atttps://github.com/longld/peda.git ~/peda
$ echo "source ~/peda/peda.py" >> ~/.gdbinit
```

安装插件后汇编指令格式改为了intel模式，不太习惯，改变为AT&T格式：

```sh
# 改为AT&T格式
gdb-peda$ set disassembly-flavor att

# 改为Intel格式
gdb-peda$ set disassembly-flavor intel
```

**参考**：[gdb-peda安装](https://blog.csdn.net/counsellor/article/details/81290335)

此外，GDB dashboard插件也不错：[链接](https://github.com/cyrus-and/gdb-dashboard)

# 3 文档

* [《100个gdb小技巧》](https://wizardforcel.gitbooks.io/100-gdb-tips/content/index.html)
* [gdb手册](https://sourceware.org/gdb/current/onlinedocs/gdb/)

# 4 GDB卡片

<img src="https://s2.loli.net/2022/05/27/joJaNUkKdB5Pnew.png" width = "1000" height = "800" alt="图片名称" align=center id=204 />

<img src="https://s2.loli.net/2022/05/27/yrgOVkfGmJQxjDT.png" width = "1000" height = "800" alt="图片名称" align=center id=205 />

# 5 常见指令





# X 参考

* [gdb基本命令(非常详细)](https://blog.csdn.net/tianya_lu/article/details/123648314)
* [EAX、ECX、EDX、EBX寄存器的作用](https://www.cnblogs.com/qq78292959/archive/2012/07/20/2600865.html)
* [X86汇编入门-寄存器32位 - 弦外之音](https://www.xianwaizhiyin.net/?p=1035)
