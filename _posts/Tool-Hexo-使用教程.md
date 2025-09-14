---
title: '[Tool] Hexo 使用教程'
categories:
  - - Tool
    - Hexo
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-13 23:37:08
tags:
---

# 环境准备

```shell
➜  nvm --version
0.39.1
➜  node -v
v24.8.0
➜  npm -v
11.6.0

# 安装 Hexo
npm install -g hexo-cli 
```

# 本地运行 Hexo

1. 初始化 hexo，生成 hexo-blog 目录

   ```shell
   hexo init hexo-blog
   ```

2. 进入 hexo-blog 目录，下载 pure 主题和 source 文章

   ```shell
   cd hexo-blog

   git clone https://github.com/Dragonliu2018/hexo-theme-pure.git themes/pure

   # 删除默认生成的 _posts 目录
   rm -rf source/_posts
   git clone https://github.com/Dragonliu2018/hexo-source.git source
   ```

3. 配置 pure，[ref link](https://github.com/Dragonliu2018/hexo-theme-pure)

   ```shell
   npm install hexo-wordcount --save
   npm install hexo-generator-json-content --save
   npm install hexo-generator-feed --save
   npm install hexo-generator-sitemap --save
   npm install hexo-generator-baidu-sitemap --save
   npm install hexo-neat --save
   npm install hexo-translate-title --save
   npm un hexo-renderer-marked --save
   npm i hexo-renderer-markdown-it-plus --save
   npm install hexo-deployer-git --save
   ```

4. 修改 _config.yml

   ```yaml
   # 页面中分类等导航词的语言
   - language: en
   + language: zh-CN
   
   # 搜索界面的分类部分拼接 url
   - url: http://example.com
   + url: https://dragonliu2018.github.io
   
   # 设置主题
   - theme: landscape
   + theme: pure
   
   # 设置部署
   deploy:
     type: git
     repo: https://github.com/dragonliu2018/dragonliu2018.github.io.git
     branch: master
   ```

5. 运行 hexo

   ```
   hexo g
   hexo s
   ```

# 部署 Github
> 参考：https://hexo.io/docs/one-command-deployment

1. 安装 hexo-deployer-git

   ```shell
   npm install hexo-deployer-git --save
   ```
2. 修改项目根目录 blog 下的 `_config.yml`

   ```yaml
   deploy:
     type: git
     repo: https://github.com/dragonliu2018/dragonliu2018.github.io.git
     branch: master
   ```
3. 部署

   ```
   hexo g
   ```

# 常用命令

```shell
hexo clean
hexo g
hexo s
hexo d
```
