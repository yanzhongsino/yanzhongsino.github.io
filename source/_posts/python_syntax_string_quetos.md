---
title: python脚本中字符串内的引号需要转义
date: 2024-06-05
categories: 
- programming
- python
- syntax
- string
tags:
- python
- syntax
- error
- quotes
- string

description: 记录使用python脚本时发现的一个语法错误。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2132217277&auto=1&height=32"></iframe></div>

# 记录使用python脚本时发现的一个语法错误

1. python脚本的语法错误的报错记录

```log
File "/soft/Cp-hap/scripts/parse.py", line 86
    o.write('Haven't detected reads which are long enough to support the structures')
                   ^
SyntaxError: invalid syntax
```

2. python脚本的单引号
- 这样的报错是因为，在python脚本中，用单引号引起来的字符串中间不能再有单引号了，而脚本parse.py中错误的使用了**Haven't**在单引号内。
- 修改的办法很简单，把**Haven't**改成**Have not**；或者在字符串中的单引号前面加上转义字符反斜杠，改为**Haven\'t**；或者把字符串外面的单引号改成双引号，改为**"Haven't detected reads which are long enough to support the structures"**。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>