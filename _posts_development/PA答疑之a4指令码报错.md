---
title: PA答疑之a4指令码报错
tags:
  - PA
categories:
  - 计组
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-27 13:06:36
---

# 1 问题

PA3.1在运行/bin/bmptest时，遇到提示a4这条指令没有实现的情况：

<img src="https://s2.loli.net/2022/05/27/yOt4KiAgJuleBNQ.png" width = "500" height = "400" alt="图片名称" align=center id=202 />

# 2 解决

## 2.1 法1

在 navy-apps/Makefile.compile 修改o2为o0即可，这是关闭了代码优化，避免了⼀些数据未保存。

在navy-apps里make clean后再重新编译。

## 2.2 法2

实现a4指令：（`胡宇轩学弟`提供）

```c++
make_EHelper(movsb){
  vaddr_write(cpu.edi,1,vaddr_read(cpu.esi,1));
  t0 = 1;
  rtl_add(&cpu.edi,&cpu.edi,&t0);
  rtl_add(&cpu.esi,&cpu.esi,&t0);
  print_asm("movbx");
}
```

DF等于0是递增，等于1是递减，具体实现见386手册：

<img src="https://s2.loli.net/2022/05/27/AimEsOoM9XalY8c.png" width = "600" height = "400" alt="图片名称" align=center id=203 />
