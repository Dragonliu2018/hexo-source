---
title: Python读取xlsx表格文件失败
tags:
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-20 23:46:50
---

# 1 报错

报错信息如下：

```sh
xlrd.biffh.XLRDError: Excel xlsx file; not supported
```

# 2 解决

## 2.1 调低xlrd版本（推荐）

xlrd过高，卸载旧版本重新安装1.2.0：

| 版本     | 支持                    |
| -------- | ----------------------- |
| xlrd 1.2 | 支持 .xls 、 .xlsx 文件 |
| xlrd 2.0 | 只支持.xls文件          |

在cmd中执行：

```sh
pip uninstall xlrd
pip install xlrd==1.2.0
```

再运行代码出现下面的报错：

```sh
AttributeError: 'ElementTree' object has no attribute 'getiterator'
```

在新版python3.9中，windows中使用的更新删除了getiterator方法，所以我们老版本的xlrd库调用getiterator方法时会报错。

1. 找出Python安装目录`python\Lib\site-packages\xlrd`下的`xlsx.py`文件；
2. 如果找不到文件地址，可以在 cmd 输入 `pip show xlrd` 即可找到xlrd的文件地址；
3. 将两个地方的`getiterator()`改成`iter()`；
4. 然后重新载入程序就解决了。

## 2.2 调低excel版本

excel文件的版本过高，复制源文件，另存为：xls格式。

# X 参考

* [xlrd.biffh.XLRDError: Excel xlsx file； not supported，两种解决方案](https://blog.csdn.net/sinat_37868031/article/details/113376079?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-113376079-blog-111904690.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-113376079-blog-111904690.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=1)
* [AttributeError: ‘ElementTree‘ object has no attribute ‘getiterator‘](https://blog.csdn.net/suhao0911/article/details/110950742)
