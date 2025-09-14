---
title: Python-类-使用简易教程
tags:
  - 待补充
categories:
  - [Python, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-02-11 14:07:06
---

# 1 模板

```python
class Action(object):
    # 类属性
    is_fail = False

    def __init__(self, sid, fid):
        self.id = sid  # 唯一标志信息
        self.fid = fid  # 父进程唯一标志信息
        
    def get_id(self):
        return self.id

    def get_fid(self):
        return self.fid

test_action = Action(sid=sid, fid=fid)
print(test_action.get_id())
print(Action.is_fail)
```

更多参考：[菜鸟教程](https://www.runoob.com/python3/python3-class.html)
