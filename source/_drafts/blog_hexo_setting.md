---
title: 
date: 2022-04-11
categories:
- blog
- markdown
tags:


description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2117115&auto=1&height=32"></iframe></div>

# hexo+next支持公式语法

公式语法：在公式两侧加上美元符号，两侧各一个美元符代表行内公式，各两个美元符代表独立行的公式。

默认的hexo部署的markdown不支持公式语法，设置后可支持。

设置：
1. 安装插件`npm install hexo-math --save`
2. 在hexo设置文件_config.yml 中添加 hexo-math 插件

```
markdown:
  plugins:
    - markdown-it-footnote
    - markdown-it-sup
    - markdown-it-sub
    - markdown-it-abbr
    - markdown-it-emoji
    - hexo-math
```

3. 打开 主题 的 mathjax 开关

打开 theme/next/_config.yml 文件，找到mathjax 位置, 设置为以下

```
# MathJax Support
mathjax:
  enable: true
  per_page: true
  cdn: //cdn.bootcss.com/mathjax/2.7.1/latest.js?config=TeX-AMS-MML_HTMLorMML
# Han Support docs: https://hanzi.pro/
```

4. 在需要公式支持的博客中打开mathjax开关

```
---
title: A Title
date: 2020-02-08
tags:
- tag1
- tag2
categories:
- parent
- child

mathjax: true
---
```

# 文件名大小写导致的404
## 问题
1. 用`hexo g -d`部署博客test.md到github pages上之后，正常显示。
2. 如果这时把博客文件名test.md改成Test.md（修改大小写字母），再`hexo g -d`部署，则会在博客网页显示此篇博客，但点进此篇博客内容，则会显示404网页。

## 原因
原因是git命令默认配置中忽略了文件名的大小写。

## 解决方案
1. 修改git的配置文件。
2. 修改`.deploy_git/.git/config`文件中`ignorecase = true`为`ignorecase = false`
3. 删除`.deploy_git/`下所有文件，只保留`.deploy_git/.git/`文件夹
4. push到github仓库

```
git commit -m "clean"
git push
```

5. 重新部署博客
  
```
hexo clean
hexo g -d
```

# references
1. hexo设置markdown公式支持：https://zhuanlan.zhihu.com/p/105986034
2. 文件名大小写导致的404：https://trifond.github.io/2018/12/05/hexo-folder-ignore-case/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>