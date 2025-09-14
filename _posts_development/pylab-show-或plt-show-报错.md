---
title: pylab.show()或plt.show()报错
tags:
 - matplotlib
categories:
  - [环境与工具]
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-22 20:04:21
---

# 1 pylab.show()问题

最近使用实验室的机子进行模型训练，**发生报错**：

```sh
XIO:  fatal IO error 0 (Success) on X server "localhost:13.0"
      after 355 requests (355 known processed) with 2 events remaining.
```

**定位代码**：

```sh
pylab.show()
```

**原因**：机子没有界面，导致显示失败。

**解决**：只需将图片保存，而非显示。

```sh
# pylab.show()
pylab.savefig('./data/net_pic/textcnn_100_0.001_300.jpg')
```

# 2 plt.show()问题

之前一个同学也遇到了类似的报错：

```sh
UserWarning: Matplotlib is currently using agg, which is a non GUI backend, so cannot show the figure.
plt.show()
```

**定位代码**：

```sh
plt.show()
```

**解决方法**：

```sh
# plt.show()
plt.savefig('./data/net_pic/textcnn_100_0.001_300.jpg')
```

# X 参考

* [python matplotlib 画图保存图片简单例子](https://blog.csdn.net/m0_37052320/article/details/79640467)
