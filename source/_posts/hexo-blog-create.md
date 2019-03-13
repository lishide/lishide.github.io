---
title: 使用 hexo 搭建部署 GitHub 博客
date: 2018-02-12 11:37:27
tags: hexo
---
# 环境配置

- Node.js
- Git
- Hexo

<!--more-->

## 安装 Node.js
进入[Node.js 官网](https://nodejs.org/en/)，下载对应版本，一路安装即可。
## 安装 Git
[Git](https://git-scm.com/download)，下载后直接安装即可。
## 安装 Hexo
当 Node.js 和 git 安装好，下面就需要安装 Hexo 了，终端执行以下命令：

```bash
npm install -g hexo-cli
```
## Hexo 安装测试
测试一下，输入命令，显示版本号即表明安装成功。
```bash
hexo -v
```
以上软件安装好，就已经生成了一个博客系统，只不过这是最基础也是最简洁的博客系统。

## Hexo 初始化
创建一个 **hexo（或 blog）** 文件夹，终端 cd 到该文件夹下，执行命令：
```bash
hexo init
```

开启 hexo 服务器
```bash
hexo s
```

此时，浏览器中打开网址：[http://localhost:4000](http://localhost:4000)，能看到 Hexo 默认的星空主题的网页。

# 关联 GitHub
## 创建 GitHub 账号并新建项目
项目名称为`用户名.github.io`的固定写法，如下图所示
> 注意：`用户名`就是以后博客地址的一部分，所以起一个自己喜欢的有个性的名字。

## 配置 Hexo 生成的配置文件

拷贝 GitHub 项目地址链接，终端 cd 到本地 hexo 文件夹下，打开文件夹下 `_config.yml` 文件，在最后增加如下配置：

```bash
deploy:
    type: git
    repository: https://github.com/用户名/用户名.github.io.git
    branch: master
```

> 注意：在编辑所有的 `_config.yml` 文件时（包括主题 theme 里的 `_config.yml` 文件）在所有的冒号`:`后边都要加一个空格，否则执行 hexo 命令会报错。

在 **hexo** 文件夹目录下执行生成静态页面命令：
```bash
hexo g
```

> 此时若出现如下报错：
ERROR Local hexo not found in ~/hexo
ERROR Try runing: 'npm install hexo --save'
则执行命令：
npm install hexo --save
若无报错，自行忽略此步骤。

再执行配置命令：
```bash
hexo d
```

若你未关联 GitHub，则执行 hexo deploy 命令时终端会提示你输入 GitHub 的用户名和密码，即
```bash
Username for 'https://github.com':
Password for 'https://github.com':
```
按照提示输入用户名和密码。

hexo deploy 命令执行成功后，浏览器中打开网址 [http://你的用户名.github.io](http://你的用户名.github.io) 能看到和打开 [http://localhost:4000](http://localhost:4000) 时一样的页面，这时别人也可以访问了。


# 日常更新博客（以后每次部署文章的步骤）

```bash
//新建文章
$ hexo new ”postName”
//清除缓存文件 (db.json) 和已生成的静态文件 (public)
$ hexo clean
//生成缓存和静态文件
$ hexo g
//重新部署到服务器（可以先执行 hexo s 命令，打开本地 hexo 服务器，在本地预览修改完之后在执行 hexo d 部署到服务器）
$ hexo d
```

# 安装主题 theme
可以到[Hexo官网主题](https://hexo.io/themes/)页去搜寻自己喜欢的 theme。目前我跟着别人的教程，安装的是**yilia**，以后可能会换。

终端 cd 到 hexo 文件夹下，执行如下命令：

```bash
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
打开 **hexo** 文件夹下的 theme，发现 `yilia` 就在该目录下了。我们默认使用的是 `landscape`。
修改 **hexo** 文件夹下的 `_config.yml` 文件。将 theme 的名称 `landscape` 修改为 `yilia`。

终端 cd 到 hexo 目录下执行日常更新博客用的命令，即：
```bash
$ hexo clean
$ hexo g
$ hexo d
```

至于更改 theme 内容，比如名称、描述、头像等去修改**hexo/_config.yml**文件和**hexo/themes/yilia/_config.yml**文件中对应的属性名称即可，不要忘记冒号:后加空格。

# 后记

以上便是我搭建我的个人博客的全过程，希望对大家有所帮助。本文也参考了多位前辈的经验，作为自己的一个理解和操作过程，分享了出来，这里希望大神不要介意，感谢前辈的分享。

Have a nice day.
