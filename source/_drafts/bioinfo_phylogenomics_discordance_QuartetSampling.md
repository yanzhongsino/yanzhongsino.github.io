---
title: 用软件Quartet Sampling来计算分支支持率，评估系统发育树不一致程度
date: 2022-11-12
categories: 
- bio
- bioinfo
tags: 
- phylogenomics
- phylogenetic discordance
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

#### Quartet Sampling（QS）method
周秋杰的蜂斗草族的系统文章用到这个方法评估系统发育树的不一致：Out of chaos: Phylogenomics of Asian Sonerileae：https://www.sciencedirect.com/science/article/pii/S1055790322001944#b0290




对snp、scg和plastome数据都跑

nohup quartet_sampling.py --tree ../species.tre--align melastoma.species29.snp.miss9d1043mac3a2.min4.phy.varsites.phy --reps1000 --threads 4 --lnlike 2  --verbout > snp.log 2>&1 &


速度很快，6Mb多的snp数据40分钟跑完。

# references
1. Quartet Sampling软件github：https://github.com/fephyfofum/quartetsampling
2. paper：https://bsapubs.onlinelibrary.wiley.com/doi/full/10.1002/ajb2.1016

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>


