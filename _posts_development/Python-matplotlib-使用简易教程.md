---
title: Python-matplotlib-使用简易教程
date: 2022-01-17 09:44:22
tags:
  - matplotlib 
categories:
    - [环境与工具]
    - [Python, 基础]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
toc: true #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
---

# 1 基础知识

[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.lines.Line2D.html)

- 画板figure，画纸Sublpot画质，可多图绘画
- 画纸上最上方是标题title，用来给图形起名字
- 坐标轴Axis，横轴叫x坐标轴label，纵轴叫y坐标轴ylabel
- 图例Legend 代表图形里的内容
- 网格Grid，图形中的虚线，True显示网格
- 点 Markers：表示点的形状。

# 2 绘图步骤

```python
#导入matplotlib的pyplot模块
import matplotlib.pyplot as plt

#1 基础绘图
#第1步：定义x和y坐标轴上的点   x坐标轴上点的数值
x=[1, 2, 3, 4]
#y坐标轴上点的数值
y=[1, 4, 9, 16]
#第2步：使用plot绘制线条第1个参数是x的坐标值，第2个参数是y的坐标值
plt.plot(x,y)
#第3步：显示图形
plt.show()


#2 定义绘图属性
'''
color：线条颜色，值r表示红色（red）
marker：点的形状，值o表示点为圆圈标记（circle marker）
linestyle：线条的形状，值dashed表示用虚线连接各点
'''
plt.plot(x, y, color='r',marker='o',linestyle='dashed')
#plt.plot(x, y, 'ro')
'''
axis：坐标轴范围
语法为axis[xmin, xmax, ymin, ymax]，
也就是axis[x轴最小值, x轴最大值, y轴最小值, y轴最大值]
'''
plt.axis([0, 6, 0, 20])
plt.show()


#3 给图片提阿甲注释和标题等

#第1步：定义x和y坐标轴上的点  x坐标轴上点的数值
x=[1, 2, 3, 4]
#y坐标轴上点的数值
y=[1, 4, 9, 16]

#第2步：使用plot绘制线条 第1个参数是x的坐标值，第2个参数是y的坐标值
# 折线图
plt.plot(x,y)
# 柱形图
# plt.barh(x,y)
#添加文本 #x轴文本
plt.xlabel('x坐标轴')
#y轴文本
plt.ylabel('y坐标轴')
#标题
plt.title('标题')
#添加注释 参数名xy：箭头注释中箭头所在位置，参数名xytext：注释文本所在位置，
#arrowprops在xy和xytext之间绘制箭头, shrink表示注释点与注释文本之间的图标距离

plt.annotate('我是注释', xy=(2,5), xytext=(2, 10),
            arrowprops=dict(facecolor='black', shrink=0.01),
            )

#第3步：显示图形
plt.show()
```

# 3 多图绘制

```python
# 柱形图
fig, axs = plt.subplots(nrows=2, ncols=2, figsize=(20, 12), dpi=100)

axs[0][0].set_title('HeatLoadLight')
axs[0][0].set_xlabel('DateTime')
axs[0][0].set_ylabel('LoadValue')
axs[0][0].bar(range(12), list, fc='g')

axs[0][1].set_title('HeatLoadHeavy')
axs[0][1].set_xlabel('DateTime')
axs[0][1].set_ylabel('LoadValue')
axs[0][1].bar(range(12), list, fc='g')

axs[1][0].set_title('CoolLoadLight')
axs[1][0].set_xlabel('DateTime')
axs[1][0].set_ylabel('LoadValue')
axs[1][0].bar(range(12), list, fc='g')

axs[1][1].set_title('CoolLoadHeavy')
axs[1][1].set_xlabel('DateTime')
axs[1][1].set_ylabel('LoadValue')
axs[1][1].bar(range(12), list, fc='g')

fig.autofmt_xdate()
plt.show()

# 折线图
fig, axs = plt.subplots(nrows=1, ncols=2, figsize=(20, 6), dpi=100)

axs[0].set_title('xxx')
axs[0].set_xlabel('Hour')
axs[0].set_ylabel('Value')
axs[0].plot(range(168), Q_trans_light, c='red', label='Light')
axs[0].plot(range(168), Q_trans_heavy, c='blue', label='Heavy')
axs[0].legend(loc='best')

axs[1].set_title("xxx")
axs[1].set_xlabel('Hour')
axs[1].set_ylabel('Value')
axs[1].plot(range(168), Q_trans_light, c='red', label='Light')
axs[1].plot(range(168), Q_trans_heavy, c='blue', label='Heavy')
axs[1].legend(loc='best')

fig.autofmt_xdate()
plt.show()
```

# X 参考

* [菜鸟教程——Matplotlib 柱形图](https://www.runoob.com/matplotlib/matplotlib-bar.html)

