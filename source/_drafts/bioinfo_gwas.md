---
title: gwas
date: 2023-03-22
categories: 
- bioinfo
- R
- plot
tags:
- 
description: 用plink做GWAS分析
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>


# 分析前 
## 数据
1. 表型数据：sample.pheno
- 文本文件，包含三列
- 第一列是家系(Family ID, FID)，第二列是个体(Individual ID, IID)，第三列是表型值。

```
A1	A1	dark
A2	A2	dark
A3	A3	yellow
A4	A4	yellow
```

2. 变异数据：sample.vcf.gz
- 一般用保存了全基因组snps的vcf格式文件
- 前期需要进行snp的filter，保留高质量的snp

## 准备文件
### 将vcf文件转换成plink可读文件格式（map，ped）
1. 命令
- `vcftools --gzvcf sample.vcf.gz --plink --out sample`
2. 结果
- 生成sample.map和sample.ped文件
3. notes 
- **可能的问题**：- 如果运行vcftools时遇到`Unrecognized values used for CHROM: HiC_scaffold_2 - Replacing with 0`的提示信息，会使得输出的文件sample.map丢失染色体信息，即sample.map文件的第一列全部变成0（本该是染色体ID）。
- **可选的解决方案**：1. 创造染色体映射文件，映射文件包含两列染色体ID：`bcftools view -H sample.vcf | cut -f 1 | uniq | awk '{print $0"\t"$0}' > chrom-map.txt`；2. 使用创造的染色体映射文件做映射去运行vcftools：`vcftools --gzvcf sample.vcf.gz --plink --chrom-map chrom-map.txt --out sample`

### 用plink将map和ped文件转换成plink二进制格式（bed,bim,fam）
1. 命令
- `plink --file sample --make-bed --allow-extra-chr --out sample2`
2. 参数
- `--file sample`：指定map和ped格式文件的前缀
- `--make-bed`：指生成plink二进制格式文件
- `--allow-extra-chr`：指允许数字以外的染色体ID格式
- `--out sample2`：指定输出文件后缀。
3. 结果
- sample2.bed
- sample2.bim
- sample2.fam
- sample2.nosex
- sample2.log

### 候选等位基因列表创建
`zcat sample.vcf.gz | awk 'BEGIN{FS="\t";OFS="\t";}/#/{next;}{{if($3==".")$3=$1":"$2;}print $3,$5;}'  > alt_alleles`

# GWAS关联分析
1. 命令
- `nohup plink --bfile sample2 --make-pheno sample.phen "yellow" --assoc --adjust --reference-allele alt_alleles --allow-extra-chr --allow-no-sex --out sample_assoc &`
2. 参数
- `--bfile sample2`：指定plink二进制文件的前缀
- `--make-pheno sample.phen "yellow"`：指定表型文件和表型值为yellow的是case组，另一种表型值dark则为control组
- `--assoc`：不允许有协变量
- `--logistic`：使用逻辑回归分析（适合质量性状），允许有协变量，速度比`--assoc`慢；结果里可能会有NA，需要手动去除。
- `--linear`：使用线性回归分析（适合数量性状），允许有协变量，速度比`--assoc`慢
- `--adjuct`：生成多种类型的矫正p值，避免假阳性
- `--covar`：指定协变量文件
- `--reference-allele alt_alleles`：指定候选等位基因列表文件
- `--allow-extra-chr`：允许数字以外的染色体ID格式
- `--allow-no-sex`：允许不指定性别的样品
- `--out sample_assoc`：指定输出文件的前缀
3. 结果
- sample_assoc.assoc
- sample_assoc.assoc.adjusted
- sample_assoc.log
- sample_assoc.nosex




# 可视化/画图
## 曼哈顿图和QQ图

## 用qqman

```R
install.packages("qqman")
library("qqman")
data<-read.table("sample_assoc.assoc",header=T)
head(data)
qq(data$P) # 绘制qq图（如果用调整过的P值，这里的P换成对应的调整过P值的列名）
manhattan(data,chr="CHR",bp="BP",p="P") # 绘制manhattan图，阈值线默认是5和8
```

## 用CMplot
```R
install.packages("CMplot")
library("CMplot")
data<-read.table("sample_assoc.assoc",header=T)
head(data)
CMplot(data,plot.type="q",threshold=0.05) # 绘制qq图
CMplot(data,plot.type = "m", threshold = c(0.01,0.05)/nrow(dat), threshold.col=c('grey','black'), threshold.lty = c(1,2), threshold.lwd = c(1,1), amplify = T,signal.cex = c(1,1), signal.pch = c(20,20),signal.col = c("red","orange")) # 绘制manhattan图，阈值线默认是5和8

```

# references

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>
