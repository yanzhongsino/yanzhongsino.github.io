---
title: 假基因鉴定
date: 2022-08-22
categories:
- bioinfo
tags:
- pseudo genes


description: 
---


## PSMC分析
通过二倍体基因组上存在的变异位点推断分化时间，通过分化时间的分布推断有效群体大小的历史动态。

三百万年那次高峰，有可能是人类从猩猩中分化出来时期的群体规模高峰期。不过，这么久远的分化区域意味着高度差异的等位基因片段（高度杂合）。这些高度杂合的片段也有可能来自平衡选择，或重测序比对过程中重复序列不正确比对产生的错误SNP。因此，三百万年的高峰可能并不可信。

psmc的参数
-t： maximum 2N0 coalescent time; the upper limit of the TMRCA
-r: the initial θ/ρ value ; initial theta/rho ratio

psmc_plot.pl的参数
-u 4.79e-9: absolute mutation rate per nucleotide [2.5e-08]，碱基突变率，默认是2.5*10-8次方subst./syn. site/year。
-g 1:number of years per generation [25],物种平均世代间隔，默认人类的25年。


蕨类植物的同义突变率:
4.79 x 10^-9 subst./syn. site/year :the first estimate of fern nuclear genome evolutionary rates with polypodiaceous nuclear genomes (Barker 2009)

荷叶铁线蕨的物种平均世代间隔约为1年。


time bwa mem -t 12 ../genome/Adiantum.reniforme.fa Adiantum.reniforme_R1.clean.u.fq Adiantum.reniforme_R2.clean.u.fq | samtools sort -@ 12 -m 12G > Adiantum.reniforme.sort.bam #把二代数据mapping到基因组上
samtools mpileup -C50 -uf ../genome/Adiantum.reniforme.fa Adiantum.reniforme.sort.bam | bcftools view --threads 16 -O z -o Adiantum.reniforme.vcf.gz # call snp，得到的是genotype likelihoods的vcf文件
bcftools call Adiantum.reniforme.vcf.gz -c -v --threads 16 -O z -o Adiantum.reniforme.variants.vcf.gz # 生成只有snp和indel的vcf文件
bcftools view -H Adiantum.reniforme.variants.vcf.gz | wc -l # 并统计snp数量，结果是6729595个snp，即6.73Mbp，基因组大小6.27Gbp（6272116485），杂合度0.1073%。
vcfutils.pl vcf2fq -d 8 -D 50 | pigz -p 12 > diploid.fq.gz # 生成二倍体fas文件

samtools mpileup -C50 -uf ../genome/Adiantum.reniforme.fa Adiantum.reniforme.sort.bam | bcftools call --threads 16 -c - | vcfutils.pl vcf2fq -d 8 -D 50 | pigz -p 12 > diploid.fq.gz #以上两步可合并

fq2psmcfa -q 20 diploid.fq.gz > diploid.psmcfa #格式转换
splitfa diploid.psmcfa > split.psmcfa #若需要做bootstrap需先把太长序列分割
nohup psmc -N25 -t15 -r5 -p "6+25*2+6+8" -o diploid.psmc diploid.psmcfa & #psmc计算二倍体间序列差异，-p参数为psmc有效群体大小变化的图随时间变化划分的阶段数量，例如默认的-p "4+25*2+4+6"表示从古自今依次经历了一个4单位+25个2单位+一个4单位+一个6单位的时期。数字越大需要的计算资源越多。
for i in {1..100};do nohup psmc -N25 -t15 -r5 -b -p "6+25*2+6+8" -o round${i}.psmc split.psmcfa & #-b参数表示bootstrap抽样做psmc，重复100次。估计服务器运算能力，可以10个重复作为一批。
cat diploid.psmc round*.psmc >combine.psmc #psmc结果合并
psmc_plot.pl -p -u 4.79e-9 -g 1 combine combine.psmc #画图，生产combine.eps和combine.pdf。
-p指用epstopdf生成pdf文件，相当于运行完多加一个`epstopdf combined.eps`一步。
-g为该物种繁殖一代的时间，比如人的默认为25年，该处值为-g 25；荷叶铁线蕨为1年。
-u为该物种碱基替换率，参考已有文献对研究类群的替换率的推断，比如蕨类4.79e-9，也可由进化树的枝长除以r8s评估该物种的分歧时间得到。


# references
1. 

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>