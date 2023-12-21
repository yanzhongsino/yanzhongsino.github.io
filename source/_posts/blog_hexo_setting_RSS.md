---
title: hexo博客下打开和设置RSS订阅
date: 2023-12-21
categories:
- blog
- hexo
- setting
- RSS
tags:
- RSS
- hexo
description: hexo博客下打开和设置RSS订阅，是在NexT主题下设置的。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1985419342&auto=1&height=32"></iframe></div>

# hexo博客下打开和设置RSS订阅的步骤

1. 在项目根目录下，使用git bash窗口安装hexo-generator-feed插件
- `npm install hexo-generator-feed --save`
2. 修改项目根目录下的设置文件_config.yml，添加以下内容：

```
Plugins: hexo-generator-feed # 添加插件
# Feed Atom
feed:
    type: atom
    path: atom.xml
    limit: 20
    hub:
    content:
    content_limit: 140
    content_limit_delim: " "
    order_by: -date
    icon: /images/favicon_200x200.png
```

3. 修改主题根目录下的设置文件_config.yml，在social和follow_me节点下修改或添加以下内容：

```
social:
    RSS: /atom.xml || fa fa-rss # 启用RSS图标
follow_me:
    RSS: /atom.xml || fa fa-rss # 开启RSS订阅入口
```

4. 重新部署博客网站，即可看到侧边栏出现RSS订阅入口，示例可看我的博客网站：https://yanzhongsino.github.io/

```
hexo clean
hexo generate
hexo deploy
```

# references
1. hexo博客NexT主题设置RSS订阅：https://tanwucheng.github.io/2021/03/03/4.Hexo%E5%8D%9A%E5%AE%A2NexT%E4%B8%BB%E9%A2%98%E8%AE%BE%E7%BD%AERSS%E8%AE%A2%E9%98%85/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>