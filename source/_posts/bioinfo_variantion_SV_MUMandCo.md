---
title: 结构变异分析软件：MUMandCo
date: 2022-07-17
categories: 
- bioinfo
- variantion
- structural variation
tags: 
- MUMandCo
- structural variation
- SV
- insertion
- deletion
- tandem duplication
- tandem contraction
- inversion
- translocation

description: 介绍了分析两个基因组间的结构变异的软件MUMandCo，用它检测插入，缺失，倒位，易位，串联重复复制，串联重复收缩等结构变异。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe></div>

# 1. MUMandCo简介
MUMandCo是用于**两个基因组间**的**结构变异（structural variant，SV）**检测分析的软件，基于MUMmer(v4)的nucmer比对结果检测全基因组范围内的插入，缺失，串联重复，倒位和易位等结构变异。

2020年发表在Bioinformatics。

# 2. MUMandCo原理
MUMandCo调用MUMmer的nucmer模块做互换的全基因组比对（whole-genome alignmetn，WGA），delta-filter模块做全局（g-）和多对多（m-）的过滤，show-coords模块解析坐标。

1. 选出与参考基因组有最精确、非重叠的alignments的query contigs。其余的alignment都不考虑。
2. 利用这些alignments统计超过50bp的SV的类型和特征。
3. 全局（g-）比对 基于多个contig-chromosome pairings来检测易位（translocation）片段，基于alignment orientation检测大的倒位（inversions），基于alignment gaps检测可能的插入缺失（insertion and deletion），利用互换的比对，gaps会在reference和query基因组间同时被考虑。
4. 多对多（m-）比对用于检测潜在的倒位（inversions）和重复（duplications）。
5. 用全局（g-）和多对多（m-）比对的过滤去除假阳性。
6. 参数`-b`可以调用BLAST来检测插入和缺失是重复的（mobile）还是新的（novel）。
7. 生成保存了reference和query基因组的SV的类型、坐标等细节的tsv文件。

# 3. MUMandCo可以检测的内容
- 插入（insertion）：>=50bp & <=150kb
- 缺失（deletion）：>=50bp & <=150kb
- 串联重复复制（tandem duplication）：>=50bp & <=150kb
- 串联重复收缩（tandem contraction）：>=50bp & <=150kb
- 倒位（inversion）：>=1kb
- 易位（translocation）：>=10kb

# 4. MUMandCo安装
- MUMandCo是个bash脚本，下载后赋予执行权限即可使用。
- 依赖的软件包括MUMmer(v4)和samtools，如果使用`-b`参数则还需要blast。

```
git clone https://github.com/SAMtoBAM/MUMandCo.git
chmod 711 ./MUMandCo/mumandco_v3.8.sh
```

# 5. MUMandCo使用
1. 运行

`bash mumandco_v3.8.sh -r genome.fa -q query.fa -g 125500000 -t 24 -b -o out`

2. 参数
- -r genome.fa：参考基因组
- -q query.fa：被检测的基因组
- -g 125500000：参考基因组大小，单位是bp
- -t 24：线程24，默认1
- -b：添加blast选项用来确认插入/缺失是mobile/repetitive还是novel。我的理解是mobile/repetitive代表别的地方有这个插入/缺失，novel代表是一个新产生的插入/缺失。
- -ml：alignments的最小长度；默认50bp
- -o out：输出文件的前缀；默认是mumandco

3. 时间
- 两个都是约300Mbp的基因组的SV分析，用了24线程，运行总耗时5小时。

# 6. MUMandCo结果
1. out.summary.txt
- 总结SV的类型和数量的文件

```
mumandco
Total_SVs	8252
Deletions	3337
Insertions	4363
Duplications	44
Contractions	8
Inversions	144
Translocations	356
```

2. out.SVs_all.tsv
- 保存了所有检测到的SVs的每一个SV的类型、坐标、长度等详细信息。
- 共有9列，包括ref_chr	query_chr	ref_start	ref_stop	size	SV_type	query_start  query_stop	info。
- 第六列SV_type包括contraction，deletion_mobile，deletion_novel，duplication，insertion_mobile，insertion_novel，inversion，transloc。
- 第九列info包含以下三种情况：
  - 'complicated': 在一个区域（region）有多个calls，一般是插入和缺失的重叠造成。
  - 'double': 在相同的坐标位置（coordinates）有几个calls; 一般是串联重复复制或者串联重复收缩有多个拷贝的改变造成。
  - ']chrX:xxxxxx]' : 标记易位片段与其他片段的关联的VCF指示符。

```
ref_chr	query_chr	ref_start	ref_stop	size	SV_type	query_start  query_stop	info
MCscaf001	Contig151	257669	257671	188	insertion_mobile	18955     19143
MCscaf001	LG12	674171	698315	10734	insertion_mobile	12964720     12975454
... ...
MCscaf254	LG05	9960	45987	36027	transloc	10452396	10497136	]MCscaf012:3067777]
MCscaf254	LG05	13407	16116	2709	deletion_mobile	10455842	10467408	complicated
```

3. out.SVs_all.withfragment.tsv
- 与out.SVs_all.tsv格式和内容类似，多一列fragment信息，保存了对应的参考基因组的碱基序列（插入的序列来自query基因组）。

4. out.SVs_all.vcf
5. out_alignments文件夹
- 保存了alignments文件，包括ref和query各自的的delta，delta_filter，delta_filter.coords，delta_filter.coordsg文件

# 7. 结果整理和统计
1. 通过out.SVs_all.tsv文件的第六列SV_type值把不同的SV类型分开，例如单独获取translocations的信息：`cat out.SVs_all.tsv| awk '$6=="transloc" {print $0}' >transloc.tsv`
2. 大部分类型都是每个一行记录，所以行数就是总数，总数与out.summary.txt文件是一致的。
3. translocations这种类型大部分有两行记录，所以行数不是总数，如果想要计算总数可以这样：`cat transloc.tsv |cut -f1-8|uniq |wc -l`。

我整理了一个这样的表格，用来一览SV的概况，供参考。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/structural_variants_sum.png?raw=true" title="结构变异概况" width="90%" align=center/>

**<p align="center">结构变异概况</p>**


感慨一句，选择什么计算机语言来写代码真没那么重要，人家bash脚本也可以发Bioinformatics。

# 8. references
1. https://github.com/SAMtoBAM/MUMandCo
2. paper: https://academic.oup.com/bioinformatics/article/36/10/3242/5756209


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>