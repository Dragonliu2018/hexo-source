---
title: 【开源代码学习--若依】vue+springboot文件下载
tags:
  - 若依
categories:
  - [Java, Spring Boot]
  - [前端, Vue]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-03-26 21:57:18
---

# 1 引入

需要文件下载功能，如下图所示：

<img src="https://s2.loli.net/2022/03/26/MiuO3xawsctC8Vg.png" width = "900" height = "300" alt="图片名称" align=center id=124 />

# 2 解决

## 2.1 手写(404，无法下载)

* [若依手册——上传下载](http://doc.ruoyi.vip/ruoyi/document/htsc.html#%E4%B8%8A%E4%BC%A0%E4%B8%8B%E8%BD%BD)

* [Java中将Map转String，String转Map](https://cloud.tencent.com/developer/article/1790636)
* [vue+springboot文件下载](https://www.cnblogs.com/javalinux/p/14807391.html)
* [使用VUE和SPRING BOOT实现文件下载](https://www.freesion.com/article/77181445797/)

## 2.2 使用官网提供的封装方法

**参考**：[使用若依前后端分离下载需要授权的url文件 ](https://blog.51cto.com/YangRoc/5057161)

1. 前端使用封装方法`this.$download.resource(resource);`：

   ```vue
   // 表格中的下载按钮
   <el-table-column label="下载报告" align="center" class-name="small-padding fixed-width">
       <template slot-scope="scope">
   		<el-button
              size="mini"
              type="text"
              icon="el-icon-download"
              @click="handleDownload(scope.row)"
              v-hasPermi="['system:report:downLoad:test']"
              >下载</el-button>
       </template>
   </el-table-column>
   
   // handleDownload(scope.row)函数实现
   /** 下载报告按钮操作 */
   handleDownload(row) {
       this.reset();
       const id = row.id || this.ids
       const resource = "/profile/upload/" + row.reportName;  // 不再设立实体类
       // 默认方法
       this.$download.resource(resource);
   },
   ```

2. 增加对xml文件的授权：修改`MimeTypeUtils.java`文件

   ```java
   public static final String[] DEFAULT_ALLOWED_EXTENSION = {
       // 图片
       "bmp", "gif", "jpg", "jpeg", "png",
       // word excel powerpoint
       "doc", "docx", "xls", "xlsx", "ppt", "pptx", "html", "htm", "txt",
       // 压缩文件
       "rar", "zip", "gz", "bz2",
       // 视频格式
       "mp4", "avi", "rmvb",
       // pdf
       "pdf",
       // xml
       "xml",
   };
   ```

3. 修改资源默认路径，资源默认路径是`D:\ruoyi\uploadPath\upload`，修改文件`RuoYi-Vue\ruoyi-admin\src\main\resources\application.yml`

   ```yml
   # 项目相关配置
   ruoyi:
     # 名称
     name: RuoYi
     # 版本
     version: 3.8.1
     # 版权年份
     copyrightYear: 2022
     # 实例演示开关
     demoEnabled: true
     # 文件路径 示例（ Windows配置D:/ruoyi/uploadPath，Linux配置 /home/ruoyi/uploadPath）
     profile: E:/05 Code/RMDVS/uploadPath
     # 获取ip地址开关
     addressEnabled: false
     # 验证码类型 math 数组计算 char 字符验证
     captchaType: math
   ```

## 2.3 补充(修改日志默认路径)

日志默认路径：`E:\home\ruoyi\logs`

修改文件：`RuoYi-Vue\ruoyi-admin\src\main\resources\logback.xml`

```xml
<!-- 日志存放路径 -->
<!--	<property name="log.path" value="/home/ruoyi/logs" />-->
<property name="log.path" value="E:/05 Code/RMDVS/logs" />
```

