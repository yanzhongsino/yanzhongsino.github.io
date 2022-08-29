---
title: 结构变异分析软件：Assemblytics
date: 2022-08-02
categories: 
- bio
- bioinfo
- variantion
- structural variation
tags: 
- structural variation
- SV
- insertion
- deletion
- tandem expansion
- tandem contraction
- repeat expansion
- repeat contraction
- biosoft
- Assemblytics

description: 介绍了分析两个基因组间的结构变异的软件Assemblytics。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe></div>

# 1. Assemblytics简介
1. Assemblytics在2016年发表在Bioinformatics上，用于检测检测基因组间的结构变异（Structural Variation，SV）。
2. Assemblytics检测的结构变异主要包括：INDELs，tandem和repeat的expansion和contraction；不包含倒位inversion和易位translocation的检测。
3. Assemblytics建议使用基因组组装的contigs而非scaffolds，避免Ns对统计结构变异产生影响（假阳性）。

<img src="https://oup.silverchair-cdn.com/oup/backfile/Content_public/Journal/bioinformatics/32/19/10.1093_bioinformatics_btw369/5/btw369f1p.gif?Expires=1662466735&Signature=XxOCD8I9su8WHNrORriUZmNxiCU219tKxNJr~iM-QEGM1pWydiON3YFnn~znOY~ibASOkh6T8rTksGBcPKFI-3ymeYLx2FDqR29phWp90a4of40ouDPqKsK5Crw5PVK5qNbbbV67n5hKpsaxUOtim-jDp5kyM8lZmPzQGm6qTRE5nclS6Lis-mQw96ZQFeguCbTS1k4Re7ULtzaMO1uLOEWG-401s~2JbBG07HI7cmt2UAP3zLV~n8CV8ISpi~W8SqvBaF7EjEqpPITGxSQQJCTP8EOg1erhp7Knf6jgTuwvnGShz4-cNNH2oRN3~2kw6aug3~B0qqPmhgFkW4JYGQ__&Key-Pair-Id=APKAIE5G5CRDK6RD3PGA" width=100% title="Assemblytics" alt="Assemblytics" align=center/>

**<p align="center">Figure 1. variant class and web results interface Assemblytics**
图片来源： [Assemblytics paper](https://academic.oup.com/bioinformatics/article/32/19/3021/2196631)</p>

# 2. Assemblytics使用
Assemblytics的使用很简单，网站也说得清楚明了。

1. 先用MUMmer把reference genome和query genome比对，得到共线性结果。

`nucmer -maxmatch -l 100 -c 500 REFERENCE.fa ASSEMBLY.fa -prefix OUT`

- -l 100：最小匹配长度设置成100
- -c 500：最小匹配序列簇的匹配长度为500
- -prefix OUT：输出结果文件前缀为OUT

2. 把delta结果文件压缩，压缩后的结果OUT.delta.gz上传到Assemblytics网站。

`gzip OUT.delta`

3. 填写参数，提交即可在线运行和得到结果。参数包括：
- Description：物种名或样品名
- Unique Sequence length required：需要的独特序列长度，代表一个决定用于call variants的序列是否独特的一个锚定（anchor），用于代替读取比对（read alignment）的映射质量的过滤，默认是10000。
- Maximum variant size：最大变异的大小，默认是10000。
- Minimum variant size：最小变异的大小，默认是50。

# 3. 结果
运行结果包括可交互的可视化图的界面和可下载的压缩包。

主要结果在文件**Assemblytics_structural_variants.summary**中，文件内容示例：

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

# 4. references
1. http://assemblytics.com/
2. https://github.com/marianattestad/assemblytics
2. paper: https://academic.oup.com/bioinformatics/article/32/19/3021/2196631

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>