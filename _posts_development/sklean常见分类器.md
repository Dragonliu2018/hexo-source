---
title: sklean常见机器学习分类器
tags:
categories:
  - 人工智能
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-16 20:28:16
---

# 1 问题

使用python的机器学习库sklearn实现常见的机器学习分类算法，如决策树、随机森林等。

# 2 代码实现

```python
import time
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
import numpy as np


def rf_train():
    api_train = np.array([[1, 2, 3], [1, 2, 2]])
    type_train = np.array([1, 0])
    api_test = np.array([[1, 2, 1]])
    type_test = np.array([0])

    clf = DecisionTreeClassifier(random_state=0)
    rfc = RandomForestClassifier(random_state=0)
    clf = clf.fit(api_train, type_train)
    rfc = rfc.fit(api_train, type_train)
    score_c = clf.score(api_test, type_test)
    score_r = rfc.score(api_test, type_test)

    print("Single Tree:{}".format(score_c), "Random Forest:{}".format(score_r))


if __name__ == '__main__':
    start = time.time()
    print('最终版-随机森林')

    end = time.time()
    print((end - start) / 60, "min")  # 秒
```

# X 参考

* [【机器学习】Sklearn 常用分类器（全）](https://blog.csdn.net/weixin_41571493/article/details/83011147)
* 
