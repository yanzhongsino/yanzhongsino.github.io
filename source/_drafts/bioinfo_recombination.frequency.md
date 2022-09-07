---
title: 线粒体的重复介导重组的鉴定和计算重组频率
date: 2022-09-02
categories: 
- bio
- bioinfo
tags: 
- mitogenome
description: 线粒体的重复介导重组的鉴定和计算重组频率。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

通常在植物线粒体上存在重复序列（这里的重复序列不是核基因组的TE等转座元件的概念，就是一段序列在另一个位置同时存在完全一致的序列），则有可能介导重组，从而生成线粒体的多种构象。如果是repeat pairs的长度小于测序reads的insert size长度（illumina PE reads是350bp左右，每对测序reads不相同），可以用reads mapping到两种构象（参考构象和重组构象）来判断repeat pairs是否介导重组，并且通过mapped reads的计数来计算重组频率（recombination frequency）。


构建参考序列和重组序列

mapping注意事项

1. mapping可以把参考序列和重组序列（一共四条）一起当作reference，避免同一reads mapping到多个构象从而重复计数的情况。
2. 如果担心叶绿体派生的reads影响重组率的计算，还可以加上叶绿体基因组，这样叶绿体的reads会被mapping到叶绿体上，从而起到过滤的作用。
3. 如果担心核基因派生的reads影响，可以用一个cutoff值(比如100bp)，小于100bp的mapping被筛除。


for i in $(ls *fa);
do
	{
		bwa index ${i}
		bwa mem -t 4 ${i} sample_1.clean.fq sample_2.clean.fq | samtools sort -@ 4 -m 4G > ${i}.bam
		samtools view ${i}.bam |awk -F"\t" '($4+$9)>411 {print $0}' > ${i}.bam.filter #筛选需要的mapped reads，否则bam文件太大，这里的`awk -F"\t"`参数设定列分隔符非常重要，已经坑过我两把了。
	} &
done

wait
echo "repeat10 done"





-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>


