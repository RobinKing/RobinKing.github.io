---
layout: post
title:  "添加多说评论系统到jekyll"
date:   2016-05-17 16:22:21
categories: [Computer Science]
tags:   jekyll
---
### 序：背景
jekyll默认使用disqus评论系统，支持的很多帐号并不适合国内使用。在俺大天朝，当然要用有本地特色的disqus了。哈哈哈～～
国内本土化的评论系统有很多，我大概瞄了一下[多说](http://duoshuo.com/)和[友言](http://www.uyan.cc/)，然后就随便选了一个(多说)～～
******
### 一、添加多说
多说的添加还是非常方便的，在[官网](http://duoshuo.com/)首页点击[我要安装](http://duoshuo.com/create-site/),登陆之后简单设置一下，就会生成各种框架的代码，俺选择将通用代码粘贴在俺的博客中。
俺先修改了`_config.yml`文件，增加`duoshuo_shortname:  robincn`字段。
然后修改博客主题，仿照`disqus`增加`_includes/duoshuo_comments.html`，文件内容如下。

最后，修改了 `_includes/scripts.html`，修改disqus附近的代码为如下版本：

这样就可以了，哈哈哈哈～～～～～
