---
title: 用PseudogenePipeline鉴定假基因
date: 2022-08-22
categories:
- bioinfo
tags:
- pseudo genes
- PseudogenePipeline


description: 
---

# 假基因

# PseudogenePipeline 简介
PseudogenePipeline是Shiulab开发的一个鉴定假基因的流程包，v1.0.0版本基于python2，v2.0.0版本基于python3。

# PseudogenePipeline 原理

# PseudogenePipeline 使用
## 软件安装
### 依赖requirements
1. python
2. RepeatMasker
3. tfasty，FASTA包 version 36及以上 [https://faculty.virginia.edu/wrpearson/fasta/]，这次用的fasta-36.3.8h版本。下载解压即可。


下载PseudogenePipeline，是python脚本包，无需安装。
https://github.com/ShiuLab/PseudogenePipeline







## pseudogene假基因注释
### 原理
在基因组未被注释到基因的区域查找与基因相似的片段。




### genome mask
TE.gff文件做genome mask，共5.12G的repeat序列被mask

### 准备tblastn.out文件
formatdb -i Adiantum.reniforme.repeatmask.fa -p F -t Adiantum.reniforme.repeatmask -o T # 建库
tblastn -db genome.mask/Adiantum.reniforme.repeatmask.fa -num_threads 4 -query ../../input/annotation/Adiantum_nelumboides.proteins_primaryTranscriptsOnly_v1.1.fa -out tblastn.out -outfmt 6 # tblastn比对。可能由于基因组大,6G，多线程会中断，只能单线程，可以把蛋白文件Adiantum_nelumboides.proteins_primaryTranscriptsOnly_v1.1.fa切分成多份，分别单线程tblastn，然后把结果合并。注意蛋白文件和基因组文件的序列id不能有空格，否则自动去除空格和空格后内容。

### 运行PseudogenePipeline
准备parameter_file
```
b_out=/disk1/zhongyan/project/20201026_Adiantum.nelumboides/output/pseudogene/tblastn.out #前面准备的tblastn.out文件，<tblastn_output_file>
p_seq=/disk1/zhongyan/project/20201026_Adiantum.nelumboides/output/pseudogene/Adiantum_nelumboides.proteins_primaryTranscriptsOnly_v1.1.nospace.fa #蛋白序列，<protein_sequence_FASTA_file>
g_seq=/disk1/zhongyan/project/20201026_Adiantum.nelumboides/output/pseudogene/genome.mask/Adiantum.reniforme.repeatmask.fa #基因组序列，<genome_sequence_FASTA_File>
b_filter=y # 是否做blastout的filter，<filter blastouput? y/n>
b_data=tabular # <define type of blastdata: tabular, blastall, blastplus>
p_codes=/disk1/zhongyan/project/20201026_Adiantum.nelumboides/output/pseudogene/PseudogenePipeline-master/_pipeline_scripts #<path to ../_pipeline_scripts>
o_codes=/disk1/zhongyan/project/20201026_Adiantum.nelumboides/output/pseudogene/PseudogenePipeline-master/_pipeline_scripts #<path to ../_pipeline_scripts>
f_dir=/home/zhongyan/software/biosoft/fasta-36.3.8h/bin #<path to FASTA package install>
f_prog=tfasty36 #<tfasty program name>
blosum=/disk1/zhongyan/project/20201026_Adiantum.nelumboides/output/pseudogene/PseudogenePipeline-master/_example_files/blosum50.matrix #<path to blosum matrilx file>
gff=/disk1/zhongyan/project/20201026_Adiantum.nelumboides/output/pseudogene/Adiantum_nelumboides.gene.exons_v1.1.gff3 #<gff_file>
repCut=300 #<RepeatMasker cutoff; integer, recommended 300>
repDiv=30 #<RepeatMakser divergence; integer, recommended 30>
```


命令
python2 PseudogenePipeline-master/_wrapper_scripts/CombinedPseudoWrapper.py parameter_file


gff文件格式要求繁琐，主要是对第九列内容的要求，参照软件给的例子进行增添修改。
格式要求：
- gene行，第九列包含Name信息，可与ID信息一致
- mRNA行，第九列包含longest=1/0信息，用于可变剪切筛选，1代表最长可变剪切，0代表非最长可变剪切。
- mRNA,exon,CDS,UTR行，第九列都包含pacid=信息，可与Parent信息一致，一个gene内部pacid相同。

6G基因组 跑了70h。

### 结果文件
- hiConf.RMfilt/hiConf.RMfilt.cdnm #假基因的位置信息，cdnm后缀是以假基因名字命名的
- fa.hiConf.RMfilt/fa.hiConf.RMfilt.cdnm #假基因的序列信息，cdnm后缀是以假基因名字命名的
- hiConf.RMfilt.cdnm.gff #假基因注释的gff文件

注释到的假基因之间是没有重叠的，一个位置的序列只能被注释为一个假基因。

共有222,372个假基因，其中长度大于300bp的为93594条。


### 检测pseudogene
看是否是pseudogene造成共线性blocks变短；
1. 分析前面MCScanX找到的共线性blocks，看blocks是否有成对出现在同一个contig上的，两对出现在同一contig上的blocks是否分别有共线性对应，结果是不存在符合条件的两对blocks。分析原因是contig的长度短造成的，contig长度在1-2Mbp，一个block含5个及以上基因，就占据了一条contig的全长，所以同一个contig上有多个block出现的概率变小。
2. 在最长的blocks里找pseudogene造成blocks内部gene无共线性的例子，即block内部无共线性的gene对应block上对称位置是否有这个gene的pseudogene存在。
在超过7个gene的blocks里找到5个blocks存在这种情况。
- Alignment 288: ctg4124-ctg7158：Ane23910-Ane23911之间有Ane41918的pseudogene(ctg4124 6381349 6381618)；Ane23909-Ane23910之间有Ane41900的pseudogene（重排）；
- Alignment 102： ctg11329-ctg6741：Ane66914-Ane66915之间有Ane38759的pseudogene；Ane66920-Ane66921之间有Ane38765的pseudogene(ctg11329        2818519 2817964)；
- Alignment 139: ctg1252-ctg4677：Ane27375-Ane27376之间有Ane07597的pseudogene；
- Alignment 422: ctg7700-ctg9700：Ane46157-Ane461158之间有Ane57929的pseudogene（重排）(ctg7700 2107090 2106980)；
- Alignment 396: ctg6709-ctg8004：Ane38599-Ane38600之间有Ane47496的pseudogene（重排）(ctg6709 1117953 1117655)；


### 计算pseudogene和真gene之间的Ks
Ks只能计算ORF区域的（即完整密码子，3倍数碱基），并遇到stop codon停止，所以要根据真基因的cds注释信息，找到比对到的pseudogene对应区域，并把真基因和pseudogene的ORF区域提取出来。若pseudogene比对到了intron区域，删除这部分，并从真基因密码子首位开始匹配pseudogene，提取完整密码子（3的倍数个碱基）；若pseudogene出现stop codon，可删除stop codon位置的密码子或者从stop codon处停止提取。

1. 根据pseudo的序列位置和对应真gene的位置信息，提取两者比对上的cds序列
拿_results/tblastn.out_parsed.4col.true.fa.RMfilt.hiConf中的一条pseudo序列举例说明位置的含义：
>Ane579290;ctg7700|2106977-2107090;113-150:114-4;1|0|0|0
GCAAATGAAGCAAGGTATTTGCGAGCTTCATTCAGTGCTTTCATGGATGTGCTTGTTTTGGCAACACGTACAGTTGAGGAGTTTGGGTCCACGTATCAGCTTGCAAGGCCC
表示在染色体ctg7700的2106977-2107090位置有一个真基因Ane579290的假基因pseudogene；
后面的113-150和114-4分别是指真基因和假基因比对上的区域；从tblastn的比对结果中来；真基因的113-150，由于用的真基因蛋白序列做tblastn，这个位置是指在Ane579290蛋白序列的位置；假基因的114-4位置是指在染色体ctg7700的2106977-2107090位置这条序列上的，114-4指第4到114位置的反向互补序列。

先从_results/tblastn.out_parsed.4col.true.fa.RMfilt.hiConf中提取出Ane579290相应的假基因pseudogene；再提取真基因Ane579290的cds序列，根据真基因的蛋白比对位113-150，通过计算113*3-2=337，150*3=450，找到cds对应的337-450位置提取出来；

2. 比对和删改
把两者序列保存在一个fasta文件，并用mafft/muscle等工具做align，以真基因序列为准，根据align结果对提取的假基因序列进行删除和修改，比如删除比对到intron的部分，若是假基因中包含indel，则在indel处找到上一个密码子的第三位，在这个位置结束；最终使得两条序列都包含3的倍数的对应密码子的orf序列；删改好后再做一次align，保存；

3. KaKs_Calculator
AXTConvertor gene_pseudo.fa gene_pseudo.axt #KaKs_Calculator的改格式工具，改比对后的fas格式为axt；数量少也可以手动改格式，axt格式包括三行，第一行两个序列ID之间用短横杠-相连，第二行第一条序列，第三行第二天序列；
KaKs_Calculator -i gene_pseudo.axt -o gene_pseudo.kaks -m PN #计算KaKs，用PN模型；
结果就在gene_pseudo.kaks文件中

| gene-pseudo              | Ka        | Ks        | Ka/Ks    |
| ------------------------ | --------- | --------- | -------- |
| Ane07597-Ane07597.pseudo | 0.166453  | 0.830702  | 0.200376 |
| Ane38759-Ane38759.pseudo | 0.0068327 | 0.0579878 | 0.11783  |
| Ane38765-Ane38765.pseudo | 0.0181923 | 0.0594878 | 0.305815 |
| Ane41900-Ane41900.pseudo | 0.356588  | 1.23697   | 0.288276 |
| Ane41918-Ane41918.pseudo | 0.0304071 | 0.033778  | 0.900203 |
| Ane47496-Ane47496.pseudo | 0.025231  | 0.210919  | 0.119624 |
| Ane57929-Ane57929.pseudo | 0.0380011 | 0.0703679 | 0.540034 |

从结果中看出，Ane38759，Ane38765，Ane579290三个基因和它们的假基因之间的Ks接近之前计算的WGD的Ks=0.06；可以作为共线性blocks内部假基因的证据；


# references
1. PseudogenePipeline github：https://github.com/ShiuLab/PseudogenePipeline

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>