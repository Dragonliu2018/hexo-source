---
title: 【开源代码学习--若依】Element显示图片
tags:
  - 若依
categories:
  - [前端, Element]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-26 19:47:40
---

# 1 引用项目内部的图片

```vue
<el-image  :src="require('@/assets/graphviz_images/'+this.img_path)" :fit="cover"></el-image>
```

# 2 参考

* [官方文档——Image 图片](https://element.eleme.cn/#/zh-CN/component/image)
* [element ui 或者 element-admin 模板 img 标签 图片不能正常显示或者资源不能正常加载问题](https://blog.csdn.net/yssa1125001/article/details/106203695)
* [element-ui Image组件中的隐藏功能el-image-viewer组件的使用](https://blog.csdn.net/qq_27702739/article/details/121213503)

