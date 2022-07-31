---
title: 博客撰写的语法和技巧
date: 2022-07-21
categories: 
- blog
tags: 
- blog
- hexo
- github
- markdown
- resume

description: 这篇博客是记录日常博客撰写过程中的语法和技巧，markdown的使用，hexo的使用，next主题的使用等。目前记录了在博客中插入图片，音乐和引用的方式。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5284273&auto=1&height=32"></iframe><music URL></div>

在根据博客[hexo建站，github.io发布，多终端同步](https://yanzhongsino.github.io/2018/06/05/blog_hexo.github/)配置了hexo网站（使用next主题）的基础上，记录了部分博客部署的设置技巧。



# 2. 小图标
博客首页的每个选项卡前的小图标可以参照这个网址自行修改：https://fontawesome.com/icons?from=io

# 添加统计
基于hexo的next主题进行设置

## 添加搜索功能
1. 安装hexo-generator-search
- 在项目根目录下运行`npm install hexo-generator-searchdb --save`

2. 更改站点配置文件./_config.yml
- 在最底部添加下面内容：

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

3. 修改主题配置文件themes/next/_config.yml

将local_search下的enable值改为true

## 添加访问量统计
1. 修改主题配置文件themes/next/_config.yml
- 在footer下添加counter项，值为true

```
footer：
  counter: true
```

2. 修改站点配置文件_config.yml或者主题配置文件themes/next/_config.yml
- 把busuanzi_count项成下面的设置：

```
busuanzi_count:
  enable: true
  total_visitors: true #统计访客数
  total_visitors_icon: fa fa-user #在访客数前的小图标
  total_views: true #统计访问数
  total_views_icon: fa fa-eye #在访问数前的小图标
  post_views: true #统计单篇文章阅读数
  post_views_icon: fa fa-eye #在单篇文章阅读数前的小图标
```

这样则会在博客的所有页面的底部（footer）展示博客的总访客数和访问数，在每篇文章的开头（header）展示单篇文章的阅读数。

## 添加字数统计和阅读时长估计
1. 安装字数统计插件
- 在项目根目录下运行`npm install --save hexo-word-counter`

2. 修改站点配置文件_config.yml或者主题配置文件themes/next/_config.yml
- 把symbols_count_time项成下面的设置：
```
symbols_count_time:
  separated_meta: true # 是否换行显示 字数统计 及 阅读时长
  item_text_post: false # 文章 字数统计 阅读时长 使用图标 还是 文本表示
  item_text_total: false # 博客底部统计 字数统计 阅读时长 使用图标 还是 文本表示
  symbols: true # 单篇文章字数统计
  time: true # 单篇文章阅读时长统计
  total_symbols: true # 所有文章总字数统计
  total_time: true # 所有文章总阅读时长统计
  exclude_codeblock: false # 统计是否排除代码块
  awl: 2 # 平均字节长度（Average Word Length）。默认为4，推荐值CN=2,EN=5,RU=6。
  wpm: 300 # 每分钟阅读字数（Words Per Minute）。默认为275，推荐值slow=200，normal=275，fast=350。
  suffix: "mins." # 后缀，时长单位mins。
```

# 3. references
1. https://mrlsm.github.io/2018/07/30/%E6%B7%BB%E5%8A%A0%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BD%91%E9%A1%B5/
2. https://zhuanlan.zhihu.com/p/525469921


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>