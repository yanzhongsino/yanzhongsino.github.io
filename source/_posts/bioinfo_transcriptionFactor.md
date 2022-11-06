---
title: 转录因子（transcription factor，TF）基础及WGD后保留的TF分析
date: 2022-10-18
categories:
- bio
- bioinfo
- transcription factor
tags:
- transcription factor
- PlantTFDB
- WGD

description: 本文记录了转录因子（transcription factor，TF）的基础知识，以及使用植物转录因子数据库PlantTFDB对WGD后保留的基因进行保留率的分析的操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105140&auto=1&height=32"></iframe></div>


# 1. 转录因子（transcription factor）
转录因子（transcription factor，TF）是一种蛋白质，它通过与特定DNA序列结合来控制遗传信息从DNA到信使RNA的转录速率。

TFs 的功能是调节——打开和关闭——基因，以确保它们在所需的细胞中在正确的时间和正确的数量表达。TF 组以协调的方式发挥作用，在整个生命过程中指导细胞分裂、细胞生长和细胞死亡；胚胎发育过程中的细胞迁移和组织；并且间歇性地响应来自细胞外的信号，例如激素。人类基因组中有多达 1600 个 TF 。转录因子是蛋白质组和调节组的成员。

# 2. 植物转录因子数据库PlantTFDB
植物转录因子数据库PlantTFDB是北京大学生物信息学中心研发的数据库和网站，目前包括165个植物物种的转录因子。

目前数据库已更新到v5.0，在网站http://planttfdb.gao-lab.org/index.php可以查看、下载和使用植物转录因子数据库。

网站的功能包括：
1. 上传核酸或蛋白质的fasta序列，在线做转录因子的注释。
2. 上传核酸或蛋白质的fasta序列，在线与数据库做blastx或blastp比对。
3. 下载特定植物的TF列表，CDS或蛋白质序列。
4. 查询特定TF和TF家族的功能描述。

# 3. 转录因子相关分析
转录因子分析可以应用的场景很多，这里介绍全基因组复制事件（WGD）后转录因子保留的分析。

## 3.1. WGD后保留TF的分析
### 3.1.1. 思路
除了直接看WGD后保留的基因中包含了什么种类和多少数量的TF外，还可以通过利用转录因子数据库PlantTFDB来做WGD后保留的每种TF的保留模式的进一步分析。

1. 参考
- paper: https://www.sciencedirect.com/science/article/pii/S1674205219303594 的 Retention Analysis of Transcription Factors部分。
- 在博客**鉴定全基因复制事件(WGD)后保留的复制基因** https://yanzhongsino.github.io/2022/10/18/bioinfo_WGD_geneRetention/ 的基础上完成WGD后保留TF的分析
2. 基本思路
- 从PlantTFDB数据库下载已有物种（比如拟南芥）的TF家族，用下载的TF家族注释orthogroups。对每个WGD事件，确认每个TF家族的保留的orthogroups的数量。
- 有些TF家族可能会被分到几个orthogroups，为了消除一个TF家族的orthogroups的大小不均的影响，文章通过标准化计算一个保留参数R值（retention value），R值用来反映WGD事件后每个TF的保留模式。
3. R值的计算公式：$$Rvalue=(Rs⁄Ts)/(Ra⁄Ta)=Rs*Ta/Ts*Ra$$，其中：
- Rs: Number of orthogroups with retention in specific TF
- Ts: Total number of orthogroups in specific TF
- Ra: Number of all TF orthogroups with retention
- Ta: Total number of TF orthogroups
- Rs/Ts: 代表在WGD后特定TF家族保留的可能性
- Ra/Ta: 代表在WGD后所有TF家族保留的可能性
- Rvalue: 用Rs/Ts比上Ra/Ta，代表相较TF家族平均水平，特定TF家族保留的可能性的高低。Rvalue越大，特定TF家族的保留率越高。

### 3.1.2. 准备文件
1. Orthogroups.txt
- Orthofinder的结果文件/path/to/OrthoFinder/Results_xx/Orthogroups/Orthogroups.txt
- Orthofinder运行时需要包含了下载TF的物种
2. dup_wgd.og
- dup_wgd.og包含了前期分析的基因复制的、涉及特定WGD保留的那些orthogroups的ID列表。
- 可以从WGD后保留基因的分析的结果文件`N5_filter_OG_dup.tsv`中提取第二列来获取：`cat N5_filter_OG_dup.tsv|cut -f2 >dup_wgd.og`。
3. 下载Ath_TF_list.txt并转化成Ath_TF_list.og

```shell
cat Ath_TF_list.txt|sed '1d'|cut -f2 >ath_2.tem # 提取第二列geneID
cat Ath_TF_list.txt |sed '1d'|cut -f3|sort|uniq >ath.tf # 提取第三列Family
for i in $(cat ath_2.tem); do echo $i >>ath.og && grep $i /path/to/OrthoFinder/Results_xx/Orthogroups/Orthogroups.txt|cut -d ":" -f1 >>ath.og ; done # 根据geneID提取orthogroups
sed -i -e '1i\Gene_ID\torthogroups_ID' -e "s/ /\t/g" ath.og # 在ath.og文件首行前插入标题行，并把列间分隔的空格改成tab分隔。
paste Ath_TF_list.txt ath.og >Ath_TF_list.og # 横向拼接Ath_TF_list.txt和ath.og两个文件
head Ath_TF_list.og && tail Ath_TF_list.og # 检查一下首尾的第二列和第四列是不是一样，看有没有拼接错误
```

### 3.1.3. 统计R相关参数
1. 这里的Ta、Ra、Ts、Rs可以用两种数量来代表，一种是统计TF_ID的数量，另一种是统计Orthogroups的数量。
- 下面的是统计TF_ID的数量，如果想要统计Orthogroups的数量，则需要在每一个值统计命令`wc -l`前面加上`cut -f5|sort|uniq|`来提取Orthogroups并去重。
2. 对每一个ath.tf里的Family，统计Ta,Ra,Rs和Ts值

```shell
for i in $(cat ath.tf);
do
    ta = $(($(cat Ath_TF_list.ogs|wc -l)-1)) # 统计Ta值（有标题行，结果需要减1）
    ra = $(grep -f dup_wgd.og Ath_TF_list.ogs|wc -l) # 统计Ra值
	rs=$(grep -f dup_wgd.og Ath_TF_list.ogs |awk -v awka="$i" '$3 == awka {print$0}'|wc -l);
	ts=$(awk -v awka="$i" '$3 == awka {print $0}' Ath_TF_list.ogs |wc -l);
	echo "${i} ${rs} ${ts} ${ra} ${ta}" >> ath_r.tem
done
```

3. 有了Ta,Ra,Rs和Ts值，接下来就可以计算Rvalue=(Rs⁄Ts)/(Ra⁄Ta)了。
- `cat ath_r.tem|sed "s/ /\t/g"|awk -F"\t" '{print $0,($1*$4)/($2*$3)}'|sed '1i\TF\tRs\tTs\tRa\tTa\tRvalue' >ath_r.txt` 待检查是否有效

### 3.1.4. 绘制热图
热图绘制可以参考博客https://yanzhongsino.github.io/2022/11/06/R_plot_heatmap

- 如果只有一次WGD的TF保留结果，可以直接根据Rvalue判断哪些TF家族保留率高。
- 如果有多次WGD的TF保留结果，或者做了多个物种的TF数据库保留结果，可以绘制热图相互比较。

用R包pheatmap绘制热图，简单快捷。(notes: 画热图这里的代码还需根据数据格式调整)

```R
df<-read.table("tf_rvalue.txt",sep= " ", header = T,row.names = 1)
df_row <- hclust(dist(df)) #对行聚类
df <- df[df_row$order,] #按行聚类结果排序
df_column <- hclust(dist(t(df))) #对列聚类
df <- df[,df_column$order] #按列聚类结果排序

BiocManager::install("pheatmap")
library(pheatmap)
pheatmap(df,color = colorRampPalette(c("lightgreen", "yellow","orange","red"))(20),legend_breaks = c(1:4), legend_labels = c("1.0","2.0","3.0","4.0"), border_color="white",treeheight_row = 50, treeheight_col = 8, display_numbers = TRUE, number_color = "black",main = "TF heatmap",cellwidth = 50, cellheight = 10)
# 其中color = colorRampPalette(c("lightgreen", "yellow","orange","red"))(20) #设置颜色渐变，值从低到高依次是浅绿色-黄色-橙色-红色，共20个颜色。
```

# 4. references
1. wiki:transcription factor: https://en.wikipedia.org/wiki/Transcription_factor
2. PlantTFDB: http://planttfdb.gao-lab.org/index.php
3. paper: https://www.sciencedirect.com/science/article/pii/S1674205219303594

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>