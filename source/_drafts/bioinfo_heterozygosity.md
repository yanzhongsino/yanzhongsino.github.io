---
title: 
date: 2022-11-11
categories: 
- bio
- bioinfo
tags: 
- mitogenome
- repeat
- recombination
- rearrangement
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

基于多组学的vcf格式文件，计算杂合度（heterozygosity）和近交系数（inbreeding coefficient）

vcftools --gzvcf sample.vcf.gz --het --out sample
生成sample.het，包含四列:

```
INDV	O(HOM)	E(HOM)	N_SITES	F
sampleID_01	390045	374779.2	400619	0.59079
```

计算每个个体的杂合度和近交系数 (F) 可以快速找到异常个体，例如 自交（强负 F），存在高测序错误问题或被另一个个体的 DNA 污染导致杂合性膨胀（高 F），或 PCR 重复或低深度导致等位基因丢失，从而低估杂合性（强负 F）。 但是，请注意，这里我们假设 Hardy-Weinberg 平衡。 如果个体不是从同一人群中抽样，则由于 Wahlund 效应，预期的杂合性将被高估。 即使样本来自多个群体，计算杂合性仍然值得，以检查是否有任何个体脱颖而出，这可能表明存在问题。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=30% title="wechat_public_QRcode.png" align=center/>


