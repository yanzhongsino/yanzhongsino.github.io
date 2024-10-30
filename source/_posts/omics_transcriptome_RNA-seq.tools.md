---
title: RNA-sequencing数据分析工具比较
date: 2021-11-19 20:50:00
categories: 
- omics
- transcriptome
tags:
- RNA-sequencing
- RNA-seq
- HISAT2
- StringTie
- Oases
- LoRDEC
- STARlong
- Salmon-SMEM
- DESeq2
- GATK
- Trinity
- 
description: RNA-sequencing数据分析工具的比较，学习文章内容的笔记。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=176028&auto=1&height=32"></iframe></div>

# 1. background
2017年nature communications上发表的Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis，对RNA-sequencing(RNA-seq)数据的39种分析工具做了比较和总结。

RNA-seq分析流程主要有四大模块：
- RNA-seq变异分析(Genomic variants)
- 短读长数据的亚型检测(Short-read isoform detection)
- 长读长数据的亚型jiance(Long-read isoform detection)
- 表达分析(Expression analysis)

<img src="https://media.springernature.com/full/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig1_HTML.jpg?as=webp" title="RNACocktail分析协议" width="90%" />

**<p align="center">Figure 1. RNACocktail分析协议**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

# 2. 比较结果总结
1. 比对工具：**HISAT2**在准确性和运算速度上表现最好值得推荐，STAR和TopHat在剪切位点总数上表现更好。
2. 有参组装工具：**StringTie**在组装的转录本数量，转录本水平的准确性和运行速度上都表现最好，Cufflinks表现一般，isoform detection and prediction(IDP)在基因水平准确性上表现最好。
3. 从头组装工具：**Trinity**转录本长，灵敏度高；SOAPdenovo-Trans转录本多，准确性高；**Oases**鉴定长转录本有优势，能够较好地涵盖到低表达的基因。
4. 三代测序错误纠正工具：**LoRDEC**在纠错质量和速度上更有优势，LSC在纠正后reads比对率的改善上表现更好。
5. 全长转录本亚型检测工具：注重质量选GMAP，注重速度选**STARlong**。
6. 转录本的定量：**Salmon-SMEM**(不经过比对)运行速度和表现都更好；StringTie(基于基因组)的定量结果更接近于不基于基因组比对的工具结果。
7. 差异表达的工具：**DESeq2**在各项得分中均优于其他的工具。
8. 检测基因组和转录组的突变：**GATK**在突变检测方面具有较高的准确性，在运行时间方面GATK与Samtools没有明显的差异。

作者总结了每一步的高精度工具，作为RNA-seq分析工具选择的一般建议。

<img src="https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig8_HTML.jpg?as=webp" title="RNACocktail计算流程" width="90%" />

**<p align="center">Figure 2. RNACocktail计算流程**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

# 3. 比较结果
## 3.1. 比对工具
### 3.1.1. 比对工具用法
RNA-seq分析的第一步通常是转录本的鉴定，需要把RNA-seq reads比对到合适的参考序列上。

如果用基因组作为参考序列可以检测到新的转录本，但可能需要耗费更多的计算资源；如果用转录组作为参考则无法找出新的转录本，但速度更快。如果研究物种没有可靠的参考序列，可以重头组装对转录本进行鉴定。

比对工具可以把RNA-seq short reads往参考基因组上进行mapping，做比对(alignment)，进行剪切点预测(junction prediction)和exon-intron边界的预测。

### 3.1.2. 比对结果
比较了三款比对主流软件，**HISAT2**在准确性和运算速度上表现最好值得推荐，STAR和TopHat在剪切位点总数上表现更好。
- **HISAT2**在所有样品中能检测到的剪接位点验证率最高，但找到的剪接位点总数则比TopHat和STAR都要少。在运行速度方面，HISAT2则有较大的优势，比STAR快大约2.5倍并且比TopHat快大约100倍。
- **STAR**对于成对reads的唯一比对表现则比较好，特别是对于有较长读长的MCF7-300细胞系的数据，不过STAR会有更多含有soft-clipped和碱基错配的较低质量比对情况。
- 如果单看有soft-clipped的reads，**TopHat**会把这部分reads全部舍弃。

<img src="https://media.springernature.com/full/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig2_HTML.jpg?as=webp" title="比对工具比较结果" width="90%" />

**<p align="center">Figure 3. 比对工具比较结果**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

## 3.2. 有参组装工具
比对完成后，接着组装转录本。比较了三款组装工具，StringTie在组装的转录本数量，转录本水平的准确性和运行速度上都表现最好，Cufflinks表现一般，isoform detection and prediction(IDP)在基因水平准确性上表现最好。
- 转录本数量：StringTie>Cufflinks~IDP
- 基因水平准确性和灵敏度：IDP>Cufflinks>StringTie
- 转录本水平准确性：StringTie>IDP>Cufflink
- 运行速度：StringTie>Cufflinks~IDP
- IDP则会忽略单外显子转录本

<img src="https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig3_HTML.jpg?as=webp" title="有参组装工具比较结果" width="90%" />

**<p align="center">Figure 4. 有参组装工具比较结果**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

## 3.3. 从头组装工具
比较了三款从头组装转录本的工具：Trinity转录本长，灵敏度高；SOAPdenovo-Trans转录本多，准确性高；Oases鉴定长转录本有优势，能够较好地涵盖到低表达的基因。
- Trinity：组装的转录本较长，灵敏度高。
- SOAPdenovo-Trans：组装到转录本数量多，准确性最高，对于高表达的转录本有明显的组装偏好性；花费的内存和CPU最低。
- Oases：在所有样品中都有较好的N10-N50长度的表现，鉴定长转录本有优势；能够较好地涵盖到低表达的基因。

<img src="https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig4_HTML.jpg?as=webp" title="从头组装工具比较结果" width="90%" />

**<p align="center">Figure 5. 从头组装工具比较结果**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

## 3.4. 三代测序错误纠正工具
LoRDEC在纠错质量和速度上更有优势，LSC在纠正后reads比对率的改善上表现更好。
- LoRDEC：纠正后的reads质量更高，运行速度比LSC快100倍。
- LSC：纠正后的reads比对率提高更多。

## 3.5. 全长转录本亚型检测工具
注重质量选GMAP，注重速度选STARlong。
- GMAP：比对到参考序列的reads数量更多。
- STARlong：运行速度快68倍。

## 3.6. 转录本的定量
### 3.6.1. 转录本定量软件
1. 基于比对的转录本定量
- 传统方法是将read比对(spliced -aligned)到参考基因组，然后利用Cufflinks和StringTie进行转录本组装，最后进行定量。
- 如果具有参考转录本序列，reads可以直接跟转录本序列比对(aligned)，然后使用Salmon-Aln和eXpress进行定量。
2. 不经过比对(alignment-free)的转录本定量
- 主要提供了四个工具：Sailfish、Salmon-SMEM、quasi-mapping和kallisto。
3. 基于长读长技术的IDP（使用不同的短读长和长读长比对工具）

### 3.6.2. 比较结果
- StringTie的定量结果更接近于不基于基因组比对的工具结果。
- 基于转录组比对的两款工具则和Salmon-SMEM的相关性更高，考虑到Salmon-SMEM更快的运行速度，eXpress和Salmon-Aln可能使用体验不如Salmon-SMEM。
- kallisto和Salmon-SMEM表现最好。
- 基于比对基因组的工具则能够看到，使用HISAT2或者TopHat对具有较长读长的样品进行比对所得到的定量结果要比STAR要好。

<img src="https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig5_HTML.jpg?as=webp" title="转录本表达定量比较结果" width="90%" />

**<p align="center">Figure 6. 转录本表达定量比较结果**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

## 3.7. 差异表达的工具
比较了基于计数的工具（DESeq2、limma和edgeR）和基于组装的工具（Cuffdiff和Ballgown）。

**DESeq2**在各项得分中均优于其他的工具，而Cuffdiff和Ballgown的表现则相对比较差。
总的来说，基于计数的工具要比基于组装的工具效果要好；另外从运行时间上看的话，Cuffdiff也是最慢的工具。

<img src="https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig6_HTML.jpg?as=webp" title="差异表达比较结果" width="90%" />

**<p align="center">Figure 7. 差异表达比较结果**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

## 3.8. 检测基因组和转录组的突变
两款突变检测的工具：GATK HaplotypeCaller和Samtools mpileup。

**GATK**在突变检测方面具有较高的准确性，在运行时间方面GATK与Samtools没有明显的差异。

<img src="https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41467-017-00050-4/MediaObjects/41467_2017_50_Fig7_HTML.jpg?as=webp" title="突变检测比较结果" width="90%" />

**<p align="center">Figure 8. 突变检测比较结果**
from [paper: Gaining comprehensive biological insight into the transcriptome by performing a broad-spectrum RNA-seq analysis](https://www.nature.com/articles/s41467-017-00050-4)</p>

# 4. 39个RNA-seq分析工具版本号、重要参数及下载地址
## 4.1. 比对工具 —— Reference-based transcript identification
1. TopHat2： –no-coverage-search
http://ccb.jhu.edu/software/tophat/index.shtml
2. STAR: -twopassMode Basic –outFilterType BySJout
https://github.com/alexdobin/STAR/releases
3. HISAT2 2.0.1-beta –dta (or –dta-cufflinks)
http://www.ccb.jhu.edu/software/hisat/index.shtml
4. RASER 0.52 -b 0.03
https://www.ibp.ucla.edu/research/xiao/RASER.html

## 4.2. 有参考转录本组装工具
1. Cufflinks 2.2.1 –frag-bias-correct
http://cole-trapnell-lab.github.io/cufflinks/
2. StringTie 1.2.1 -v -B
http://www.ccb.jhu.edu/software/stringtie/

## 4.3. 无参考转录本组装工具
1. SOAPdenovoTrans 1.04 -K 25
https://github.com/aquaskyline/SOAPdenovo-Trans/
2. Oases 0.2.09 (Velvetv1.2.10) (velveth haslength: 25) (velvetg options: -read trkg yes)
http://www.ebi.ac.uk/~zerbino/oases/
3. Trinity 2.1.1 –normalize reads 
http://trinityrnaseq.sourceforge.net/

## 4.4. 三代长read分析工具
1. LoRDEC 0.6 -k 23 -s 3 
http://atgc.lirmm.fr/lordec/
2. GMAP 12/31/15 -f 1
http://research-pub.gene.com/gmap/
3. STARlong 2.5.1b
https://github.com/alexdobin/STAR/releases

Followed the recommended options : 
- –outSAMattributes NH HI NM MD
- –readNameSeparator space
- –outFilterMultimapScoreRange 1
- –outFilterMismatchNmax 2000
- –scoreGapNoncan -20
- –scoreGapGCAG -4
- –scoreGapATAC -8
- –scoreDelOpen -1
- –scoreDelBase -1
- –scoreInsOpen -1
- –scoreInsBase -1
- –alignEndsType Local
- –seedSearchStartLmax 50
- –seedPerReadNmax 100000
- –seedPerWindowNmax 1000
- –alignTranscriptsPerReadNmax 100000
- –alignTranscriptsPerWindowNmax 10000
- –outSAMstrandField intronMotif
- –outSAMunmapped Within

4. IDP 0.1.9 
https://www.healthcare.uiowa.edu/labs/au/IDP/

## 4.5. 定量工具
1. eXpress 1.5.1 (bowtie2 v2.2.7) (bowtie2 options: -a -X 600 –rdg 6,5 –rfg 6,5 –score-min L,-.6,-.4 –no-discordant –no-mixed)
https://pachterlab.github.io/eXpress/index.html
2. kallisto 0.42.4 
http://pachterlab.github.io/kallisto/about.html
3. Sailfish 0.9.0 
http://www.cs.cmu.edu/~ckingsf/software/sailfish/
4. Salmon-Aln 0.6.1 
https://github.com/COMBINE-lab/salmon
5. Salmon-SMEM 0.6.1
https://github.com/COMBINE-lab/salmon
index: –type fmd 
quant: -k,19 
6. Salmon-Quasi 0.6.1 
https://github.com/COMBINE-lab/salmon
index: –type quasi -k 31 
7. featureCounts 1.5.0-p1 -p -B -C
http://subread.sourceforge.net/

## 4.6. 差异表达分析工具
1. DESeq2 1.14.1 
http://bioconductor.org/packages/release/bioc/html/DESeq2.html
2. edgeR 3.16.5
http://www.bioconductor.org/packages/release/bioc/html/edgeR.html
3. limma 3.30.7 
http://bioconductor.org/packages/release/bioc/html/limma.html
4. Cuffdiff 2.2.1
–frag-bias-correct –emit-count-tables
http://cole-trapnell-lab.github.io/cufflinks/
5. Ballgown 2.6.0 
https://github.com/alyssafrazee/ballgown
6. sleuth 0.28.1
https://github.com/pachterlab/sleuth

## 4.7. 变异分析工具
1. SAMtools 1.2 (bcftools v1.2) 
samtools mpileup -C50 -d 100000
https://github.com/samtools/samtools
2. bcftools filter -s LowQual -e ‘%QUAL<20 —— DP>10000’
https://github.com/samtools/bcftools
3. GATK v3.5-0-g36282e4 (picard 1.129)
https://software.broadinstitute.org/gatk/download/
Picard AddOrReplaceReadGroups: SO=coordinate
Picard MarkDuplicates: CREATE INDEX=true VALIDATION STRINGENCY=SILENTGATK
SplitNCigarReads: -rf ReassignOneMappingQuality -RMQF 255 -RMQT 60
-U ALLOW N CIGAR READSGATK 
HaplotypeCaller: -stand call conf 20.0
-stand emit conf 20.0 -A StrandBiasBySample
-A StrandAlleleCountsBySampleGATK 
VariantFiltration: -window 35 -cluster 3 -filterName FS -filter
“FS >30.0” -filterName QD -filter “QD <2.0”

## 4.8. RNA编辑
1. GIREMI 0.2.1
https://github.com/zhqingit/giremi
2.  Varsim 0.5.1
https://github.com/bioinform/varsim

## 4.9. 基因融合
1. FusionCatcher 0.99.5a beta 
https://github.com/ndaniel/fusioncatcher
2. JAFFA 1.0.6 
https://github.com/Oshlack/JAFFA
3. SOAPfuse 1.27
http://soap.genomics.org.cn/soapfuse.html
4. STAR-Fusion 0.7.0 
https://github.com/STAR-Fusion/STAR-Fusion
5. TopHat-Fusion 2.0.14
http://ccb.jhu.edu/software/tophat/fusion_index.shtml
s

# 5. references
1. paper：https://www.nature.com/articles/s41467-017-00050-4
2. https://mp.weixin.qq.com/s/hp_wxiLDI9EVB8Z3L1bxbw
3. https://www.cnblogs.com/wangprince2017/p/9959008.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>