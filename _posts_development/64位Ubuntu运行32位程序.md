---
title: 64位Ubuntu运行或调试32位程序
tags:
  - Ubuntu
categories:
  - 计组
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-26 13:20:32
---

# 1 问题

最近在做lab2，使用64位的Ubuntu发现无法调试和运行：

```sh
➜  lab ./bomb 
zsh: no such file or directory: ./bomb
```

# 2 解决

ubuntu 64位下可以兼容32位程序的运行，但是必须要有32位基础库的支持才行。下面进行安装：

```sh
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386
sudo apt install lib32z1
```

安装完成后可以运行和调试了：

```sh
➜  lab ./bomb 
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
```

```sh
➜  lab gdb ./bomb
GNU gdb (Ubuntu 9.2-0ubuntu1~20.04.1) 9.2
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./bomb...
(gdb) 
```

# X 参考

* [ubuntu 64下运行32位程序](https://zhuanlan.zhihu.com/p/357653128)
