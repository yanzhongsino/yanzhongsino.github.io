---
title: 用miniprot做蛋白序列和基因组序列间的比对
date: 2024-09-30
categories: 
- bioinfo
- alignment
- prot
tags: 
- miniprot
- align
description: 记录了用软件miniprot进行蛋白序列和基因组序列间的比对。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=397952&auto=1&height=32"></iframe></div>

# 1. miniprot介绍
需要进行蛋白序列与基因组序列的比对，可以用李恒团队开发的miniprot。

# 2. miniprot安装
1. conda安装：`conda install miniprot`
2. 下载安装：仅依赖zlib

```shell
git clone https://github.com/lh3/miniprot
cd miniprot && make
```

# 3. miniprot使用
基因组genome.fna和蛋白序列protein.faa进行比对
## 3.1. 先建索引，再比对【推荐】
1. 建立索引：会将基因组genome.fna翻译成氨基酸序列，费时。
- `miniprot -d genome.mpi genome.fna`
2. 比对：翻译的基因组氨基酸序列与指定的蛋白序列进行比对
-`miniprot --gff -It16 --gff genome.mpi protein.faa > aln.gff`
## 3.2. 建索引和比对一步完成
- `miniprot -Iut16 --gff genome.fna protein.faa > aln.gff`

# references
1. github：https://github.com/lh3/miniprot

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>