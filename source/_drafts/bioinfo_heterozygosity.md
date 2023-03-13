---
title: 使用pixy计算群体的pi，dxy，fst
date: 2023-02-24
categories:
- bio
- bioinfo
tags:
- pixy
- pi
- dxy
- fst

description: 
---

<div align="middle"><music URL></div>

基于多组学的vcf格式文件，计算杂合度（heterozygosity）和近交系数（inbreeding coefficient）

vcftools --gzvcf sample.vcf.gz --het --out sample
生成sample.het，包含四列:

```
INDV	O(HOM)	E(HOM)	N_SITES	F
sampleID_01	390045	374779.2	400619	0.59079
```

计算每个个体的杂合度和近交系数 (F) 可以快速找到异常个体，例如 自交（强负 F），存在高测序错误问题或被另一个个体的 DNA 污染导致杂合性膨胀（高 F），或 PCR 重复或低深度导致等位基因丢失，从而低估杂合性（强负 F）。 但是，请注意，这里我们假设 Hardy-Weinberg 平衡。 如果个体不是从同一人群中抽样，则由于 Wahlund 效应，预期的杂合性将被高估。 即使样本来自多个群体，计算杂合性仍然值得，以检查是否有任何个体脱颖而出，这可能表明存在问题。


1. 获取all-sites的vcf文件
```
gatk --java-options "-Xmx32g" HaplotypeCaller -R ref.fa -I input.bam -O output.g.vcf -ERC GVCF # 生成单个样本gvcf文件
gatk --java-options "-Xmx32g" GenomicsDBImport -V output1.g.vcf -V output2.g.vcf ... --genomicsdb-workspace-path <allsamples_genomicsdb>
for chr in $(cat chr.list);do gatk --java-options "-Xmx32g" GenotypeGVCFs -R ref.fa -V samples.g.vcf -L ${chr} -all-sites -O ${chr}_allsites.vcf.gz;done  # joint-calling这里一定要加参数-all-sites(把non-variant位点也保存到vcf文件中，后面才能进行pixy的分析）。必须使用-L指定染色体，否则有报错风险。耗时步骤。
ls *_allsites.vcf.gz >vcf.list # 把生成的染色体的vcf文件名保存到vcf.list
gatk MergeVcfs -I vcf.list -O samples_allsites.vcf.gz # 合并所有染色体vcf文件到一个vcf文件，耗时步骤
```

2. 分别获取invariant和variant位点，过滤后，再合并
```
vcftools --gzvcf samples_allsites.vcf.gz --max-maf 0 --minQ 30 --max-missing 0.9 --min-meanDP 10 --max-meanDP 43 --remove filter.indv --recode --stdout | bgzip -c > samples_invariant_filtered.vcf.gz & # 提取invariant位点并过滤，同时生成out.log名称的log文件
vcftools --gzvcf samples_allsites.vcf.gz --mac 1 --min-alleles 2 --max-alleles 2 --minQ 30 --max-missing 0.9 --min-meanDP 10 --max-meanDP 43 --remove filter.indv --recode --stdout | bgzip -c > samples_variant_filtered.vcf.gz & # 提取variant位点并过滤，同时也生成out.log名称的log文件，最好与上一步骤分开在两个文件夹运行
tabix samples_invariant_filtered.vcf.gz
tabix samples_variant_filtered.vcf.gz
bcftools concat --allow-overlaps samples_variant.vcf.gz samples_invariant.vcf.gz -O z -o samples_allsites_filtered.vcf.gz # 合并invariant和variant位点到一个文件
```

3. pixy计算dxy，pi和fst

`nohup pixy --stats pi fst dxy --vcf sample_allsites_filtered.vcf.gz --populations popfile.txt --window_size 10000 --n_cores 16 &`

popfile.txt文件包含样本个体分组信息，两列内容，tab分隔。第一列样本ID，第二列分组名称（常用物种作为分组依据）。

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

## pixy的结果
根据--stats的设定，生成三个统计量的结果文件。

1. pixy_pi.txt
```
pop	chromosome	window_pos_1	window_pos_2	avg_pi	no_sites	count_diffs	count_comparisons	count_missing
BFS	HiC_scaffold_1	1	10000	0.0	6	0	6	0
AFS	HiC_scaffold_1	1	10000	0.0	6	0	6	0
... ...
```

2. pixy_fst.txt
```
pop1	pop2	chromosome	window_pos_1	window_pos_2	avg_wc_fst	no_snps
BFS	AFS	HiC_scaffold_1	30001	40000	NA	67
CFS	DES	HiC_scaffold_1	30001	40000	0.0	67
```

3. pixy_dxy.txt
```
pop1	pop2	chromosome	window_pos_1	window_pos_2	avg_dxy	no_sitescount_diffs	count_comparisons	count_missing
BFS	AFS	HiC_scaffold_1	1	10000	0.0	6	24	0
CFS	DES	HiC_scaffold_1	1	10000	0.0	6	24	0
```



-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>