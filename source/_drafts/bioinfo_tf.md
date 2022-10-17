---
title: 植物转录因子
date: 2022-07-27
categories:
- bioinfo
- transcription factor
tags:
- transcription factor
- PlantTFDB


description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2117115&auto=1&height=32"></iframe></div>


# 转录因子（transcription factor）
转录因子（transcription factor，TF）是一种蛋白质，它通过与特定DNA序列结合来控制遗传信息从DNA到信使RNA的转录速率。

TFs 的功能是调节——打开和关闭——基因，以确保它们在所需的细胞中在正确的时间和正确的数量表达。TF 组以协调的方式发挥作用，在整个生命过程中指导细胞分裂、细胞生长和细胞死亡；胚胎发育过程中的细胞迁移和组织；并且间歇性地响应来自细胞外的信号，例如激素。人类基因组中有多达 1600 个 TF 。转录因子是蛋白质组和调节组的成员。

# 植物转录因子数据库PlantTFDB
植物转录因子数据库PlantTFDB是北京大学生物信息学中心研发的数据库和网站，目前包括165个植物物种的转录因子。

目前数据库已更新到v5.0，在网站http://planttfdb.gao-lab.org/index.php可以查看、下载和使用植物转录因子数据库。

网站的功能包括：
1. 上传核酸或蛋白质的fasta序列，在线做转录因子的注释。
2. 上传核酸或蛋白质的fasta序列，在线与数据库做blastx或blastp比对。
3. 下载特定植物的TF列表，CDS或蛋白质序列。
4. 查询特定TF和TF家族的功能描述。

# WGD事件后转录因子保留的分析
转录因子分析可以应用的场景很多，这里介绍全基因组复制事件（WGD）后转录因子保留的分析。

参考paper: https://www.sciencedirect.com/science/article/pii/S1674205219303594的Retention Analysis of Transcription Factors部分。

大致的思路是：用orthofinder对基因组们找用PlantTFDB下载的拟南芥的TF的




# references
1. wiki:transcription factor: https://en.wikipedia.org/wiki/Transcription_factor
2. PlantTFDB: http://planttfdb.gao-lab.org/index.php
3. paper: https://www.sciencedirect.com/science/article/pii/S1674205219303594

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>