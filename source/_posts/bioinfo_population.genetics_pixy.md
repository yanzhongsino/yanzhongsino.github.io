---
title: 使用pixy计算群体的pi，dxy，fst
date: 2023-03-13
categories:
- bioinfo
- population genetics
tags:
- pixy
- pi
- dxy
- fst

description: 基于多组学的vcf格式文件，计算群体的pi，dxy，fst值。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105146&auto=1&height=32"></iframe></div>

# 1. 用pixy计算群体的pi，dxy，fst
pixy可以基于多组学的vcf格式文件来计算群体的pi，dxy，fst值。

## 1.1. 安装pixy

```shell
conda install --yes -c conda-forge pixy
conda install --yes -c bioconda htslib
pixy --help
```

## 1.2. 准备vcf文件
pixy需要的vcf文件与只包含snp的vcf文件不同，是同时包含变异位点（variant sites）和非变异位点（invariant sites）的全位点（all sites）文件。

1. 获取all-sites的vcf文件

```shell
gatk --java-options "-Xmx32g" HaplotypeCaller -R ref.fa -I input.bam -O output.g.vcf -ERC GVCF # 生成单个样本gvcf文件
gatk --java-options "-Xmx32g" GenomicsDBImport -V output1.g.vcf -V output2.g.vcf ... --genomicsdb-workspace-path <allsamples_genomicsdb>
for chr in $(cat chr.list);do gatk --java-options "-Xmx32g" GenotypeGVCFs -R ref.fa -V samples.g.vcf -L ${chr} -all-sites -O ${chr}_allsites.vcf.gz;done  # joint-calling这里一定要加参数-all-sites(把non-variant位点也保存到vcf文件中，后面才能进行pixy的分析）。必须使用-L指定染色体，否则有报错风险。耗时步骤。
ls *_allsites.vcf.gz >vcf.list # 把生成的染色体的vcf文件名保存到vcf.list
gatk MergeVcfs -I vcf.list -O samples_allsites.vcf.gz # 合并所有染色体vcf文件到一个vcf文件，耗时步骤
```

2. 分别获取invariant和variant位点，过滤后，再合并

```shell
vcftools --gzvcf samples_allsites.vcf.gz --max-maf 0 --minQ 30 --max-missing 0.9 --min-meanDP 10 --max-meanDP 43 --remove filter.indv --recode --stdout | bgzip -c > samples_invariant_filtered.vcf.gz & # 提取invariant位点并过滤，同时生成out.log名称的log文件
vcftools --gzvcf samples_allsites.vcf.gz --mac 1 --min-alleles 2 --max-alleles 2 --minQ 30 --max-missing 0.9 --min-meanDP 10 --max-meanDP 43 --remove filter.indv --recode --stdout | bgzip -c > samples_variant_filtered.vcf.gz & # 提取variant位点并过滤，同时也生成out.log名称的log文件，最好与上一步骤分开在两个文件夹运行
tabix samples_invariant_filtered.vcf.gz
tabix samples_variant_filtered.vcf.gz
bcftools concat --allow-overlaps samples_variant.vcf.gz samples_invariant.vcf.gz -O z -o samples_allsites_filtered.vcf.gz # 合并invariant和variant位点到一个文件
```

## 1.3. pixy计算dxy，pi和fst
1. popfile.txt群体文件
- popfile.txt文件包含样本个体分组信息，两列内容，tab分隔。第一列样本ID，第二列分组名称（常用物种作为分组依据）。

```
ERS223827   BFS
ERS223759   BFS
ERS223750   BFS
ERS223967   AFS
ERS223970   AFS
ERS223924   AFS
ERS224300   AFS
ERS224168   KES
ERS224314   KES
```

2. 运行

```shell
bgzip samples_allsites_filtered.vcf
tabix samples_allsites_filtered.vcf.gz
nohup pixy --stats pi fst dxy --vcf sample_allsites_filtered.vcf.gz --populations popfile.txt --window_size 10000 --n_cores 16 &
```

## 1.4. pixy的结果
根据--stats的设定，生成三个统计量的结果文件。

1. pixy_pi.txt：居群内的核苷酸多样性（nucleotide diversity (pi)）
- pop：popfile.txt群体文件中群体ID
- chromosome：染色体或contig的ID
- window_pos_1：基因组window的起始位置
- window_pos_2：基因组window的终止位置
- avg_pi：window的位点的平均核苷酸多样性。更具体地说，pixy 计算window中所有位点的每个位点的加权平均核苷酸多样性，其中权重由每个位点的基因分型样本数决定。
- no_sites：window中至少具有一种有效基因型的位点总数。此统计信息被提供给用户，不会直接用于任何计算。
- count_diffs：window中所有基因型之间成对差异的原始数量。这是计算 avg_pi 的分子。
- count_comparisons：window中所有基因型之间非缺失成对比较的原始数量（即比较两种基因型且均有效的情况）。这是计算 avg_pi 的分母。
- count_missing：window中所有基因型之间缺失配对比较的原始数量（即比较两种基因型且至少缺失一种的情况）。

```
pop	chromosome	window_pos_1	window_pos_2	avg_pi	no_sites	count_diffs	count_comparisons	count_missing
BFS	HiC_scaffold_1	1	10000	0.0	6	0	6	0
AFS	HiC_scaffold_1	1	10000	0.0	6	0	6	0
... ...
```

2. pixy_dxy.txt：居群间的核苷酸差异（nucleotide divergence (dxy)）
- pop1：popfile.txt群体文件中群体ID，比较的第一个居群
- pop2：popfile.txt群体文件中群体ID，比较的第二个居群
- chromosome：染色体或contig的ID
- window_pos_1：基因组window的起始位置
- window_pos_2：基因组window的终止位置
- avg_dxy：window的平均每个位点的核苷酸差异
- no_sites：window中两个群体中至少具有一种有效基因型的位点总数。此统计信息提供给用户，不会直接用于任何计算。
- count_diffs：所有基因型之间的成对交叉种群差异的原始数量。这是计算 avg_dxy 的分子。
- count_comparisons：window中所有基因型之间非缺失成对交叉种群比较的原始数量（即比较两种基因型且均有效的情况）。这是计算 avg_dxy 的分母。
- count_missing：window中所有基因型之间缺失的成对跨种群比较的原始数量（即两个基因型被比较而至少有一个缺失的情况）。这个统计数字提供给用户，不直接用于任何计算。

```
pop1	pop2	chromosome	window_pos_1	window_pos_2	avg_dxy	no_sites    count_diffs	count_comparisons	count_missing
BFS	AFS	HiC_scaffold_1	1	10000	0.0	6	24	0
CFS	DES	HiC_scaffold_1	1	10000	0.0	6	24	0
```

3. pixy_fst.txt：Weir and Cockerham’s FST
- pop1：popfile.txt群体文件中群体ID，比较的第一个居群
- pop2：popfile.txt群体文件中群体ID，比较的第二个居群
- chromosome：染色体或contig的ID
- window_pos_1：基因组window的起始位置
- window_pos_2：基因组window的终止位置
- avg_wc_fst：window的平均每个SNP（不是每个位点）的 Weir and Cockerham’s FST 
- no_snps：window内变异位点（SNPs）的总数

```
pop1	pop2	chromosome	window_pos_1	window_pos_2	avg_wc_fst	no_snps
BFS	AFS	HiC_scaffold_1	30001	40000	NA	67
CFS	DES	HiC_scaffold_1	30001	40000	0.0	67
```

# 2. references
1. pixy manual：https://pixy.readthedocs.io/en/latest/index.html
2. pixy github：https://github.com/ksamuk/pixy

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=30% title="wechat_public_QRcode.png" align=center/>