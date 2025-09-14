---
title: Hexo迁移到新电脑
date: 2021-09-21 23:28:58
tags:
- Hexo
categories: 
	- [环境与工具]
# keywords:
description:
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://i.loli.net/2021/09/21/3EG7HCh4QKFuIyV.png
toc: true  #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不
---

# 0 情景导入

换了一台电脑，将旧电脑的hexo博客写作环境迁移到新电脑上。

# 1 Git下载及配置

1. 从官网Git下载git并安装，下面设置无密push+pull；
2. 打开git bash，在用户主目录(其他目录也可)下运行：`ssh-keygen -t rsa -C "youremail@example.com"` 把其中的邮件地址换成自己的邮件地址，然后一路回车；
3. 最后完成后，会在用户主目录下生成.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH key密钥对，id_rsa是私钥，千万不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人；
4. 登陆GitHub，打开「Settings」->「SSH and GPG keys」，然后点击「new SSH key」，填上任意Title，在Key文本框里粘贴公钥id_rsa.pub文件的内容（千万不要粘贴成私钥了！），最后点击「Add SSH Key」，你就应该看到已经添加的Key。 
5. 注意：不要在git版本库中运行ssh，然后又将它提交，这样就把密码泄露出去了

# 2 安装Node.js

1. Hexo基于Node.js，Node.js下载地址：[Download | Node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/download/) 下载安装包，注意安装Node.js会包含环境变量及npm的安装，安装后，检测Node.js是否安装成功，在命令行中输入 `node -v`；
2. 检测npm是否安装成功，在命令行中输入`npm -v`

# 3 安装Hexo

新建文件夹，在此文件夹内使用npm命令全局安装Hexo，输入：

```sh
cnpm install -g hexo-cli
```

输入`hexo -v`检查是否安装成功。

> npm加速：
>
> 1. 装cnpm
>
>    ```sh
>    npm install -g cnpm --registry=https://registry.npm.taobao.org
>    ```
>
> 2. 使用cnpm
>
>    ```sh
>    cnpm install xxx
>    ```
>
> 参考：[npm使用国内镜像加速的几种方法](https://cloud.tencent.com/developer/article/1372949)

# 4 复制原电脑上的数据

## 4.1 需要复制的

* _config.yml：站点配置
* package.json：说明使用那些包
* scaffolds：文章的模板
* source：自己写的博客文件
* themes：主题
* .gitignore：限定在提交的时候哪些文件可以忽略

## 4.2 需要删除的

* .git：无论是在站点根目录下，还是主题目录下的.git文件，都可以删掉。
* node_modules：在用`npm install`会重新生成
* public：`hexo g`会重新生成
* .deploy_git：在使用hexo d时也会重新生成
* db.json文件

其实上面这些文件也就是.gitignore文件里面记载的可以忽略的内容。

# 5 npm install

在git bash中切换目录到新拷贝的文件夹里，使用`npm install` 命令，进行模块安装。很明显我们这里<font color='red'>没用hexo init初始化</font>，因为有的文件我们已经拷贝生成过来了，所以不必用hexo init去整体初始化，如果不慎在此时用了hexo init，则站点的配置文件_config.yml里面内容会被清空使用默认值，所以这一步一定要慎重，不要用hexo init。

# 6 安装其他必要组件

```sh
npm install hexo-deployer-git --save # 为了使用hexo d来部署到git上 
npm install hexo-generator-feed --save # 为了建立RSS订阅
npm install hexo-generator-sitemap --save # 为了建立站点地图
```

插件安装后，有的需要对配置文件_config.yml进行配置，具体怎么配置，可以参考上面插件在github主页上的具体说明

# 7 验证

首先本地执行，在博客根目录下执行：

```sh
hexo g
hexo s
```

此时可访问浏览器：http://localhost:4000/ ，查看是否转移成功。

接下来验证部署，执行：

```sh
hexo g
```

在线访问出错，无法访问。开始排查：

1. .deploy_git文件夹内的文件大小均为0kb，也就是生成了一些空文件；

2. <font color='pink'>public文件夹内的文件大小均为0kb，但是本地可以正常访问，不知道为啥</font>

3. 其实在执行`hexo s`时，后台存在报错

   ```sh
   INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
   (node:23336) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
   (Use `node --trace-warnings ...` to show where the warning was created)
   (node:23336) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
   (node:23336) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
   (node:23336) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
   (node:23336) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
   (node:23336) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
   ```

   解决方法：Node版本太高，切换一个低版本Node可以（我这里是14.17.6切换成12.5.0就没有这个问题了）

4. 问题已解决，可以在线访问博客了。

# 参考

* [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
* [使用hexo，如果换了电脑怎么更新博客？](https://www.zhihu.com/question/21193762)
