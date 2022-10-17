---
title: 用PSMC推断有效群体大小历史
date: 2022-10-17
categories:
- bio
- bioinfo
tags:
- biosoft
- PSMC

description: 用软件PSMC推断有效群体大小的动态历史。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2534006&auto=1&height=32"></iframe></div>

# PSMC
Pairwise Sequentially Markovian Coalescent (PSMC) 是李恒开发的一款软件，可以通过一个二倍体个体的基因组的分析获得这个个体代表的群体的有效群体大小的历史动态。

# PSMC分析原理
通过一个二倍体个体的基因组的等位基因上存在的变异位点计算序列间差异，从而推断序列的分化时间；计算所有等位基因（也可以换成等位片段）的分化时间。认为某个分化时间的等位基因数量越多，代表当时的有效群体大小越大，所以可以通过整个基因组上等位基因的分化时间的分布推断有效群体大小的历史动态。

- 这个算法让人惊叹的点在于，我们可以只从一个个体的基因组上看到群体的进化历史。
- 有意思的是，前些时候看《自私的基因》那本书的时候，书里的序也详细讲解了PSMC的算法，从等位基因回溯到基因的共同祖先，讲得很清晰，非专业人士也可以读懂，有兴趣的读者可以去看看。
- 分析人类的有效群体大小历史时，在三百万年有一次共有的高峰，有可能是人类从猩猩中分化出来时期的群体规模高峰期。不过，这么久远的分化区域意味着高度差异的等位基因片段（高度杂合）。这些高度杂合的片段也有可能来自平衡选择，或重测序比对过程中重复序列不正确比对产生的错误SNP。因此，三百万年的高峰可能并不可信。

# PSMC运行

## 文件准备
1. 基因组文件：genome.fa
2. 二代测序数据：sample_R1.clean.u.fq和sample_R2.clean.u.fq

## 运行步骤
### 准备全基因组二倍体一致序列
1. 二代序列mapping
- `time bwa mem -t 12 genome.fa sample_R1.clean.u.fq sample_R2.clean.u.fq | samtools sort -@ 12 -m 12G > sample_sort.bam` 把二代数据mapping到基因组上

2. 获取二倍体一致序列
- `samtools mpileup -C50 -uf genome.fa sample_sort.bam | bcftools view --threads 16 -c - | vcfutils.pl vcf2fq -d 10 -D 100 |pgzip -p 12 >diploid.fq.gz` 生成全基因组二倍体一致序列diploid.fq.gz
- `bcftools view`可以换成`bcftools call`
- 其中，`vcfutils.pl vcf2fq -d 10 -D 100`的`-d 10`参数是最小read depth，作者建议设置成平均depth的三分之一(1/3)；`-D 100`参数是最大read depth，建议设置成平均depth的两倍。

3. 上一步可分解为以下几步：
- `samtools mpileup -C50 -uf genome.fa sample_sort.bam | bcftools view --threads 16 -O z -o sample.vcf.gz` call snp，得到的是genotype likelihoods的vcf文件
- `bcftools call sample.vcf.gz -c -v --threads 16 -O z -o sample.variants.vcf.gz` 生成只有snp和indel的vcf文件，其中-c代表SNP calling选择consensus-caller算法，另可选-m即multiallelic-caller算法（更适合多种allel和罕见变异的情况）。
- `bcftools view -H sample.variants.vcf.gz | wc -l` 统计snp数量。snp数量除以基因组大小即可得到估计的杂合度。
- `vcfutils.pl vcf2fq -d 8 -D 50 sample.variants.vcf.gz| pigz -p 12 > diploid.fq.gz` 生成二倍体fq文件

### 运行单次PSMC
1. 格式转换
- `fq2psmcfa -q20 diploid.fq.gz > diploid.psmcfa` 格式转换

2. PSMC运行
- `nohup psmc -p "4+25*2+4+6" -t15 -N30 -r4 -o diploid.psmc diploid.psmcfa &`
- psmc计算二倍体间序列差异

3. PSMC参数
- 参数-p可根据经验或者近缘类群的文献设定，或者多尝试几个参数看看结果，-t,-N,-r可用默认参数。
- -p：代表psmc有效群体大小变化的图随时间变化划分的时间阶段单位数量，例如默认的-p "4+25*2+4+6"表示从古自今依次经历了一个4单位+25个2单位+一个4单位+一个6单位的时期。数字越大需要的计算资源越多。可根据经验调整。
- -t：maximum 2N0 coalescent time [15]; 2N0的最大世代时间，代表the upper limit of the TMRCA。
- -r: initial theta/rho ratio [4]; 起始θ/ρ率，默认4。
- -N：maximum number of iterations [30]；迭代的最大次数，默认30。
- -o：指定输出的diploid.psmcfa文件。

### 运行bootstrap的PSMC【推荐】
如果要运行多次抽样的PSMC，则在上面的格式转换后，先分割长序列，再用for循环语句运行。
1. 分割长序列
- `splitfa diploid.psmcfa > split.psmcfa` 若需要做bootstrap需先把太长序列分割

2. bootstrap抽样运行PSMC
- `for i in {1..100};do nohup psmc -b -p "6+25*2+6+8" -t15 -N30 -r4 -o round${i}.psmc split.psmcfa &` #-b参数表示bootstrap抽样做psmc，重复100次。估计服务器运算能力，可以10个重复作为一批。

3. 合并结果
- `cat diploid.psmc round*.psmc >combine.psmc` #psmc结果合并

## 画图
1. 运行
- `psmc_plot.pl -p -g 1 -u 4.79e-9 combine combine.psmc`
- 画图，生产combine.eps和combine.pdf。

2. 参数
- -p：指用epstopdf把esp转化成pdf文件，相当于运行完多加一个`epstopdf combined.eps`一步。
- -g：number of years per generation [25]，指定该物种的平均世代时间（即繁殖一代的时间），比如人的默认为25年，该处值为-g 25；许多草本植物为1年。
- -u：absolute mutation rate per nucleotide [2.5e-08]，指定该物种碱基替换率，单位是subst./syn. site/year，比如默认水稻的2.5e-08，文献里提到的蕨类4.79e-9(Barker 2009)。参考已有文献对研究类群的替换率的推断，也可由进化树的枝长除以r8s评估该物种的分歧时间得到。

# references
1. https://github.com/lh3/psmc

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>