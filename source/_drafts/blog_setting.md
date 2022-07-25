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

# 1. 添加自定义网页
添加自定义网页，自定义网页包括简历、404、About页面等，这里以添加**简历**为例。

## 1.1. 添加自定义网页
1. 在hexo/source目录下创建文件夹，例如resume。如果添加about页面就创建about文件夹。
2. 然后把网页文件index.html放在resume文件夹下。
3. 如果网页引用了css或js文件或者图片文件，也放在resume文件夹下

## 1.2. 跳过渲染
因为hexo应用的主题给每个网页都进行相同的渲染，所以需要对自定义网页设置跳过默认的渲染。

1. 在index.html设置

在网页文件index.html的开头添加下面代码设置跳过渲染的指令，这样通过`hexo g`生成网页时会跳过index.html文件，不受hexo主题渲染，而完全是一个独立的网页。

```
---
layout: false
---
```

2. 在_config.yml文件中设置

在hexo目录的_config.yml文件设置skip_render参数来设置不渲染的文件/文件夹。

- 屏蔽指定文件：`skip_render：./resume/index.html`
- 屏蔽指定文件夹的所有文件：`skip_render: ./resume/*`
- 屏蔽指定文件夹所有文件包括子文件夹和子文件夹下的文件：`skip_render: ./resume/**`
- 屏蔽多个路径：

```
skip_render:
    - ./resume/index.html
    - ./resume/image.png
```

## 1.3. 设置主页显示
在博客首页的菜单栏中增加简历项的设置

1. 在主题下的设置文件中`./themes/next/_config.yml`文件中的menu项中添加简历行`resume: /resume/index.html || fa fa-file-user`，这样可以在博客首页的菜单栏中增加简历项。
2. 其中`resume`会显示在博客主页
3. `/resume/index.html`指定自定义网页文件
4. `fa fa-file-user`定义图标，可以在https://fontawesome.com/icons?from=io选择合适的图标使用。

```
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
  resume: /resume/index.html || fa fa-file-user
```

## 1.4. 添加about页面
与上面的添加自定义页面类似：
1. 在hexo/source目录下创建文件夹about
2. 在about文件夹下创建index.md文件写入about显示内容
3. 如果使用hexo的主题显示，则无需跳过渲染
4. 把主题下的设置文件中`./themes/next/_config.yml`文件中的menu项中的about信息行的注释符#删除，表示在博客主页显示about项进入页面

# 2. 小图标
博客首页的每个选项卡前的小图标可以参照这个网址自行修改：https://fontawesome.com/icons?from=io

# 添加搜索功能
1. 安装hexo-generator-search
在项目根目录下运行`npm install hexo-generator-searchdb --save`

2. 更改站点配置文件./_config.yml

在最底部添加下面内容：

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

3. 修改主题配置文件themes/next/_config.yml

将local_search下的enable从false改为true

# 添加访问量统计


# 3. references
1. https://mrlsm.github.io/2018/07/30/%E6%B7%BB%E5%8A%A0%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BD%91%E9%A1%B5/
2. https://zhuanlan.zhihu.com/p/525469921


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>