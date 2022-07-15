---
title: 结构变异分析软件：Assemblytics
date: 2022-07-15
categories: 
- bioinfo
- variantion
- structural variation
tags: 
- Assemblytics
- structural variation
- SV
- insertion
- deletion
- tandem expansion
- tandem contraction
- repeat expansion
- repeat contraction

description: 介绍了分析两个基因组间的结构变异的软件Assemblytics。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe></div>

# 


# 结果

1. Assemblytics_structural_variants.summary

文件内容示例：

```
Insertion
                         Count       Total bp
         50-500 bp:        835          91429
     500-10,000 bp:        158         468629
             Total:        993         560058

Deletion
                         Count       Total bp
         50-500 bp:        779          85083
     500-10,000 bp:        140         289652
             Total:        919         374735

Tandem_expansion
                         Count       Total bp
         50-500 bp:         46           8957
     500-10,000 bp:        117         444964
             Total:        163         453921

Tandem_contraction
                         Count       Total bp
         50-500 bp:          2            379
     500-10,000 bp:          0              0
             Total:          2            379

Repeat_expansion
                         Count       Total bp
         50-500 bp:        373          80936
     500-10,000 bp:        968        3666087
             Total:       1341        3747023

Repeat_contraction
                         Count       Total bp
         50-500 bp:        411          92021
     500-10,000 bp:        893        3378919
             Total:       1304        3470940

Total number of all variants: 4,722
Total bases affected by all variants: 8.61 Mbp
Total number of structural variants: 4,722
Total bases affected by structural variants: 8.61 Mbp
```

# references
1. http://assemblytics.com/
2. https://github.com/marianattestad/assemblytics
2. paper: https://academic.oup.com/bioinformatics/article/32/19/3021/2196631

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>