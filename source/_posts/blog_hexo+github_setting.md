---
title: 把简历部署到网站上
date: 2022-07-31
categories: 
- blog
tags: 
- blog
- hexo
- github
- markdown
- resume
- readme
- 404 page

description: 在hexo+github.io建好博客网站的基础上，在网站中添加自定义网页（包括简历、about me、404等页面）的步骤。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5284273&auto=1&height=32"></iframe><music URL></div>

在根据博客[hexo建站，github.io发布，多终端同步](https://yanzhongsino.github.io/2018/06/05/blog_hexo.github/)配置了hexo网站（使用next主题）的基础上，记录了添加自定义网页（包括简历、about me、404等页面）的设置技巧。

# 1. 添加自定义网页
添加自定义网页，自定义网页包括简历、404、About页面等，这里以添加**简历**为例。

## 1.1. 制作简历
1. 现状
- 网上的博客在线简历相关教程大多是开发了一个专门用于简历展示的博客主题，需要博客使用这个主题以实现制作简历和在线显示的目的。
- 但我的博客用的是hexo的next主题，不想换。
- 所以最后决定用模板的html文件自己修改制作简历html。

2. 操作
- 在网上找喜欢的简历模板，提取html文件。
- 比如我的简历参考的是这个模板：https://github.com/mtics/hexo-mtics-resume
- 在模板的demo：https://mtics.netlify.app/，右键**查看网页源代码**，然后复制和保存html代码到index.html。
- 修改index.html的信息为自己的简历内容，即可使用。
- 同时为了达到同样的渲染效果，把模板https://github.com/mtics/hexo-mtics-resume/tree/master/source里的三个css文件下载下来，img/avatar.jpg文件改成自己的头像照片。

## 1.2. 添加简历
1. 在hexo/source目录下创建文件夹，例如resume。（如果添加about页面就创建about文件夹）
2. 然后把制作好的简历网页文件index.html放在resume文件夹下。
3. 简历引用了的css或js文件和图片文件，也放在resume文件夹下。

## 1.3. 跳过渲染
因为hexo应用的主题给每个网页都进行相同的渲染，所以需要对自定义网页设置跳过默认的渲染，有以下两种方法。

1. 在index.html设置
- 在网页文件index.html的开头添加下面代码设置跳过渲染的指令，这样通过`hexo g`生成网页时会跳过index.html文件，不受hexo主题渲染，而完全是一个独立的网页。

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

## 1.4. 设置主页显示
在博客首页的菜单栏中增加简历项的设置。

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

## 1.5. 简历成品
最后的效果可以参考我的简历：https://yanzhongsino.github.io/resume/index.html

# 2. 添加about页面
与上面的添加简历页面类似：
1. 在hexo/source目录下创建文件夹about
2. 在about文件夹下创建index.md文件写入about显示内容
3. 如果使用hexo的主题显示，则无需跳过渲染
4. 把主题下的设置文件中`./themes/next/_config.yml`文件中的menu项中的about信息行的注释符#删除，表示在博客主页显示about项进入页面

# 3. 添加404页面
404页面是在域名下访问不存在的地址时出现的页面。

1. 在博客主题的source下（比如我的./themes/next/source/）创建404.html文件
2. 在404.html文件中写入想要在404页面展示的内容
3. 推荐把腾讯公益404页面的代码写入404页面，为传播失踪儿童信息贡献一点微薄之力。
- 下面的**homePageUrl="https://yanzhongsino.github.io"**用于设置网站首页地址，需要改成自己的网站首页网址。
- **homePageName="返回主页（您可以进入首页继续搜索）"**设置引导词，可以替换成引导用户回到网站首页的引导语。
- 腾讯公益404页面的js地址可能会更新，可以以腾讯官方代码：https://news.qq.com/404/，发布的为准。
- 404.html示例：

```
<!DOCTYPE HTML>
<html>
<head>
  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="robots" content="all" />
  <meta name="robots" content="index,follow"/>
  <link rel="stylesheet" type="text/css" href="https://qzone.qq.com/gy/404/style/404style.css">
</head>
<body>
  <script type="text/plain" src="http://www.qq.com/404/search_children.js"
          charset="utf-8" homePageUrl="https://yanzhongsino.github.io"
          homePageName="返回主页（您可以进入首页继续搜索）">
  </script>
  <script src="https://qzone.qq.com/gy/404/data.js" charset="utf-8"></script>
  <script src="https://qzone.qq.com/gy/404/page.js" charset="utf-8"></script>
</body>
</html>
```

1. 接着正常部署博客即可。
2. 测试404页面。在网页域名下输入一个不存在的地址（比如https://yanzhongsino.github.io/111），看404页面是否更新成功。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>