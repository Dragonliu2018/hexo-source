---
title: 'PA报错之qemu: could not load PC BIOS ''bios-256k.bin'''
tags:
  - PA
categories:
  - 计组
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-10 22:24:04
---

# 1 报错

开启diff_test，nemu再make run后报错：

<img src="https://s2.loli.net/2022/05/10/mGoDxkP4RLQbIjd.png" width = "600" height = "400" alt="图片名称" align=center id=198 />

# 2 解决

下面这些链接与遇到的问题相似，**但是并未解决**

* [qemu: could not load PC BIOS 'bios-256k.bin'](https://blog.csdn.net/zhangyexinaisurui/article/details/81806572)
* [由于缺少BIOS而无法启动KVM VM](https://mlog.club/article/4454010)
* [How to Fix Error – qemu: could not load PC BIOS ‘bios.bin’?](https://techglimpse.com/qemu-system-x86-command-error-solution/)

**下面的方法解决了问题：**

自行编译一个 带i386,x86以及包含相关文件如'bios256k.bin'的qemu版本即可

参考：[QEMU虚拟机编译使用实践](https://zhuanlan.zhihu.com/p/37329713)
