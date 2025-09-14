---
title: Python-os模块-遍历文件夹下的所有文件
tags:
categories:
   - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-09 16:47:44
---

# 1 问题引入

最近在实现恶意脚本的可视化，需要实现提取文件夹下的所有文件名，实现脚本自动化。

# 2 文件夹下只有文件

当目标文件夹中只有文件时，使用os模块的`listdir()`方法即可：该方法可以返回目标路径下的文件和文件夹的名字列表，参数就是目标路径。

```python
import os

file_names = os.listdir(r'test')
print(file_names)  # ['1.txt', '2.txt', '3.txt']
```

# 3 文件夹下有文件和文件夹

当目标文件中既有文件又有文件夹时，我们使用`listdir()`方法就只能获得第一层子文件或文件夹了，而子文件夹中的内容便获取不到了。

这时候我们需要用到`os.walk()`方法：传入目标路径即可。该方法可以递归的找出目标路径下的所有文件。

```python
import os

file_names = os.listdir(r'test')
print(file_names)  # ['1.txt', '2.txt', '3.txt', 'sub-dir']

for filepath, dirnames, filenames in os.walk(r'test'):
    for filename in filenames:
        print(filename)
"""
1.txt
2.txt
3.txt
4.txt
5.txt
6.txt
"""
```

***

`os.walk()`方法可以生成三元组，也就是我们代码中的`filepath`，`dirnames`，`filenames`。我们将他们分别打印出来可以看到：其中`filepath`就是目标路径下所有文件的路径，`dirnames`是我们目标路径的所有目录名称，`filenames`则是各个路径下的文件名称列表：

```python
for filepath, dirnames, filenames in os.walk(r'test'):
    print(filepath)
"""
test
test\sub-dir
"""
for filepath, dirnames, filenames in os.walk(r'test'):
    print(dirnames)
"""
['sub-dir']
[]
"""
for filepath, dirnames, filenames in os.walk(r'test'):
    print(filenames)
"""
['1.txt', '2.txt', '3.txt']
['4.txt', '5.txt', '6.txt']
"""
```

# 4 拓展—获得目标路径下的所有文件的绝对路径

遍历目标路径下的各个文件，用`os.path.join()`方法将文件路径和文件名拼接在一起，就是各个文件的绝对路径了：

```python
for filepath, dirnames, filenames in os.walk(r'test'):
    for filename in filenames:
        print(os.path.join(filepath, filename))
"""
test\1.txt
test\2.txt
test\3.txt
test\sub-dir\4.txt
test\sub-dir\5.txt
test\sub-dir\6.txt
"""
```

# X 参考

* [python遍历文件夹下的所有文件](https://blog.csdn.net/qq_39721240/article/details/90704223)

