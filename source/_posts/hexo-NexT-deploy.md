---
title: hexo 的主题切换为 NexT 后的若干个性化配置
date: 2018-03-27 13:31:02
tags: [hexo, NexT]
---
刚使用 hexo 搭建个人博客的时候，看到推荐使用相对较多的主题是 yilia，便安装了，讲真，yilia 主题是一个简洁优雅的主题，后期也做了一些个人配置，体验上还算不错。用过一段时间后，个人对代码块的黑色背景配色有点不喜欢，而且无法自己定制，另外不支持某些鼠标手势……于是想换一款更适合自己的主题，昨天换上了 **NexT**，很清新简洁，风格是我喜欢的，哈哈~。网上有很多大佬分享了针对 **NexT** 主题的一些配置，根据个人需求，做了部分配置，本文记录将 hexo 的主题切换为 NexT 后的若干个性化配置，具体的操作就不详细描述了，主要参考了[hexo的next主题个性化教程:打造炫酷网站](https://www.jianshu.com/p/f054333ac9e6)，尊重原作，我把自己修改的地方只做序号记录即可（简称 N），以后在主题更新后方便找回自己的配置。

<!--more-->

# 使用主题
使用的 版本是`v6.0.x`，GitHub 地址：[hexo-theme-next](https://github.com/theme-next/hexo-theme-next)。

# 个性化配置
## 选择 scheme
**主题配置文件**中，设置 `scheme: Gemini`。

## 设置语言
**站点配置文件**中，设置 `language: zh-CN`。

## 设置社交账号
**主题配置文件**中，**social**节点下配置 GitHub、weibo 等账号，`||` 后面是其对应的图标。

## 修改文章内链接文本样式
参考 N5

## 修改文章底部的那个带#号的标签
参考 N6

## 在网站底部加上访问量
参考 N13

步骤：
- 打开`\themes\next\layout\_partials\footer.swig`文件
- 在 `copyright` 的 div 前添加：
```js
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```
- 在合适位置添加显示统计的代码，如下是我添加的：
```js
<div class="powered-by">
<i class="fa fa-eye"></i>
<span id="busuanzi_container_site_pv">
  访问量：<span id="busuanzi_value_site_pv"></span>
</span>
<span class="post-meta-divider">|</span>
<i class="fa fa-user-md"></i>
<span id="busuanzi_container_site_uv">
  访客数：<span id="busuanzi_value_site_uv"></span>
</span>
</div>
<span class="post-meta-divider">|</span>
```
有两种统计方式：**pv** 和 **uv**，根据个人需要选择。

添加之后进行部署，成功后刷新页面就能看到效果了。

## 网站底部字数统计
参考 N15

## 添加字数统计、阅读时长
参考 N18
参考 [Hexo 添加字数统计、阅读时长](https://sessionch.com/hexo/hexo-common-plug.html)
参考 [给hexo博客,next主题,文章添加字数和阅读时长](https://toxufe.github.io/posts/41943/)
参考 [为Hexo NexT主题添加字数统计功能](https://eason-yang.com/2016/11/05/add-word-count-to-hexo-next/)

步骤：
- 打开`\themes\next\layout\_macro/post.swig`文件
- 在 class 为 `post-mata` 的 **div** 中的添加如下内容：
```js
{% if theme.wordcount %}
<span class="post-letters-count">
  <span class="post-meta-divider">|</span>
  <i class="fa fa-file-word-o"></i>
  <span title="{{ __('post.wordcount') }}">
      {{ wordcount(post.content) }} 字
  </span>
  <span class="post-meta-divider">|</span>
  <i class="fa fa-clock-o"></i>
  <span title="{{ __('post.min2read') }}">
      {{ min2read(post.content) }} 分钟
  </span>
</span>
{% endif %}
```

# 后记
不断折腾中，不定期更新~