---
title: '[Tool][opt] Instruments + speedsope 安装与使用'
categories:
  - [Tool,opt]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-26 00:34:54
tags:
---

# A 安装引子

在 macbook 上做 cmu 15445 fall 2022 p1 时，p1 在线测试超时了，需要进行性能调优。但是 macos 上没有 perf，调研后发现替代物 → Instruments。

# 步骤

1. Instruments 集成在 xcode 中，所以先要安装 xcode（xcode 体积大，约 7G，所以一定要注意 macos 和 xcode 的版本对应 → https://developer.apple.com/cn/support/xcode/，不要下载错了版本🥦
2. 可以通过 App Store 进行下载，或者通过这个链接 → [https://developer.apple.com/download/all/?q=for Xcode](https://developer.apple.com/download/all/?q=for%20Xcode)

---

【**参考**】[**安装Xcode的方法**](https://blog.csdn.net/sunshine__sun/article/details/115062452)

# B 使用引子

Instruments 是一款苹果自带的测试工具，用于相关的性能分析和动态跟踪，其中有内存性能测试，图形性能测试、内存性能测试等。

打开 xcode；

通过导航栏找到 Instruments：

![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_08-51-01.png)

或者 `command + 空格` 直接搜索 Instruments 即可。

---

打开 `Time Profiler` 进行性能分析：

![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_08-52-01.png)

---

主要通过下面两种方式进行耗时分析：

# 1 可执行程序

选择 `可执行程序`，然后点击 `开始` 图标即可。

![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_08-52-02.png)

> ⚠️：当时分析 cum 15445 fall 2022 p1 的时候，这种不能得出好的结果，就使用了下面进程的方式。


# 2 进程

1. 这里选择分析 `所有进程`，因为要分析的是一个可执行文件，不太好把他搞成一个进程。
    ![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_08-52-03.png)
    
2. 点击 `开始` 图标，执行目标文件，等待目标文件执行完，点击 `结束` 图标；
3. 这时可以看到分析结果，目标文件也在其中：

    ![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_08-52-04.png)
    
4. 为了方便查看，可以将其单独领出来，点击右侧 ➡️；
5. 可以看到 `HardTest_4` 的主要耗时在 `HistoryCmpLess()` 函数上：

    ![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_08-52-05.png)
    
6. 耗时集中在下面三行代码，推测是 `history_` 的随机取数据导致性能低下。
    
    ![](https://raw.githubusercontent.com/Dragonliu2018/hexo-images-bed/main/2025/Snipaste_2025-09-26_08-52-06.png)
    
    查看 `history_` 类型，果然是 map 类型，但是使用 unordered_map 更加高效。更换数据类型后，耗时下降到 **7s**，符合预期。
    

# 3 火焰图

就可以利用 speedsope 生成火焰图：

[speedscope](https://www.speedscope.app/)

# X 参考

- [**Instrument工具使用**](https://juejin.cn/post/6865102561507672077)