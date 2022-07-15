---
title: 基因组质量评估：（二）用QUAST评估基因组组装
date: 2021-08-23 14:30:00
categories: 
- omics
- genome
- quality assessment
tags:
- genome
- quality assessment
- biosoft
- QUAST

description: 记录了基因组组装评估工具QUAST的安装和使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 1. QUAST是什么
QUAST(Quality Assessment Tool for Genome Assemblies)是基因组质量评估工具，通过计算各种指标来评估基因组的组装，包括N50,L50，GC含量等contig基本信息。

QUAST基于python开发，matplotlib绘图。

# 2. QUAST网站
- 主页：http://quast.sourceforge.net/
- github: https://github.com/ablab/quast

# 3. QUAST安装
1. 下载解压缩即可使用

QUAST会在第一次使用时自动编译所有子部分，因此不需要安装，解压缩即可使用。

- `wget -c https://github.com/ablab/quast/releases/download/quast_5.1.0rc1/quast-5.1.0rc1.tar.gz`
- `tar -zxvf quast-5.1.0rc1.tar.gz`
- `python quast.py --help`
- `python quast.py --version`

2. conda安装

`conda install -y quast`

3. Ubuntu 20.04系统上安装
`sudo apt-get update && sudo apt-get install -y pkg-config libfreetype6-dev libpng-dev python3-matplotlib`


# 4. QUAST使用
1. 简化版

`quast.py contigs.fas`

2. 全面版

`quast.py contigs_1.fa contigs_2.fa -r reference.fa -g genome.gff -1 reads1.fastq.gz -2 reads2.fastq.gz -o quast_out -t 12`

3. 参数
- contigs.fa是必须提供的，即等待评估组装质量的基因组，可以多个同时评估。
- -r reference.fa：参考基因组，可选；提供后有比较基因组的结果。
- -g genome.gff：参考基因组的features文件，GFF,BED等格式
- -1和-2：PE测序的FASTQ文件，可选
- -o quast_out：指定结果输出目录
- -t 12：线程
- --large：大基因组推荐加上这个参数，相当于`-e -m 3000 -i 500 -x -k --k-mer-stats`，加上这个参数后运行时间长非常多，因为有-e会做基因组的基因预测，推荐大基因组使用完整参数`-m 3000 -i 500 -x -k`来节省时间。
- -f：--gene-finding，用GeneMarkS(原核生物)或GeneMark-ES(真核生物)预测基因
- -e：即--eukaryote，默认是用GeneMarkS预测原核生物，这个参数指定基因组是真核生物，主要影响基因预测。类似的还有--fungus。还有许多与基因预测相关的参数可选。
- --rna-finding：用Barrnap预测ribosomal RNA genes
- -b：用BUSCO计算保守的orthologs数量(only on Linux)
- -m 500：小于指定长度的contig会被去除，默认是500bp。
- -i 65：小于指定长度的alignment会被去除，默认是65bp。
- -k：--k-mer-stats，基于k-mer计算质量参数，推荐用于大基因组。

# 5. QUAST结果
1. 结果文件

```
report.txt      summary table
report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)  
report.tex      Latex version
report.pdf      PDF version, includes all tables and plots for some statistics
report.html     everything in an interactive HTML file
icarus.html     Icarus main menu with links to interactive viewers
contigs_reports/        [only if a reference genome is provided]
  misassemblies_report  detailed report on misassemblies
  unaligned_report      detailed report on unaligned and partially unaligned contigs
k_mer_stats/            [only if --k-mer-stats is specified]
  kmers_report          detailed report on k-mer-based metrics
reads_stats/            [only if reads are provided]
  reads_report          detailed report on mapped reads statistics
```

看report.html网页结果全面，会有图和数据，report.txt则是主要数据。

2. 结果示例

```
cat report.txt
Assembly                    sample
# contigs (>= 0 bp)         266
# contigs (>= 1000 bp)      266
# contigs (>= 5000 bp)      186
# contigs (>= 10000 bp)     159
# contigs (>= 25000 bp)     108
# contigs (>= 50000 bp)     30
Total length (>= 0 bp)      256218469
Total length (>= 1000 bp)   256218469
Total length (>= 5000 bp)   256043090
Total length (>= 10000 bp)  255847102
Total length (>= 25000 bp)  254902657
Total length (>= 50000 bp)  252626053
# contigs                   266
Largest contig              33924140
Total length                256218469
GC (%)                      42.94
N50                         20460156
N90                         13725599
L50                         5
L90                         11
# N's per 100 kbp           66.54
```

references
1. http://quast.sourceforge.net/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>