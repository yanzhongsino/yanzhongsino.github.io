---
title: 用Trinity的脚本进行转录本定量分析
date: 2024-10-28
categories: 
- omics
- transcriptome
- express
tags:
- expression analysis
- expression abundance
- Trinity
- transcript
- align_and_estimate_abundance.pl
- RSEM
- bowtie2
description: 本文记录Trinity的转录本定量分析脚本`align_and_estimate_abundance.pl`的使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=486310963&auto=1&height=32"></iframe></div>

# 1. 转录本表达定量分析
转录本表达定量分析基本原理：通过测序获得的RNA-seq数据mapping到组装好的转录组上，通过mapped reads的数量（reads count）可以判断表达量的高低。
1. 因为reads count还受到测序深度（测序越深，count越高）和转录本长度（越短count越高）的影响，通常还需要进行标准化，消除测序深度和转录本长度的影响，标准化指标常用CPM,FPKM,TPM等。标准化后的指标可用于不同样品间的比较。
2. 如果测序获得了不同处理/不同性状的样品的数据，最常用的转录本定量分析后的运用是进行差异表达分析。通过鉴定差异表达基因来判断与性状相关的基因。

转录本表达定量分析（transcript expression quantification）又叫转录本表达丰度分析（transcript expression abundance）。

转录组本定量分析的脚本`align_and_estimate_abundance.pl`直接支持调用基于比对的定量分析软件RESE，超快非比对软件kallisto和wicked-fast软件salmon。

# 2. 软件安装
1. 脚本位置
- 安装Trinity后即拥有各种下游分析脚本，转录组本定量分析脚本`align_and_estimate_abundance.pl`位置在trinityrnaseq-v2.15.2/util/下。
2. RSEM安装
- RSEM是用于分析转录本定量表达的软件。
- 关于RSEM可参考博客[用RSEM进行转录本定量分析](https://yanzhongsino.github.io/2024/10/28/omics_transcriptome_expression_RSEM/)。

# 3. 转录本定量分析
Trinity软件自带转录组组装后的下游分析脚本，本文记录转录组本定量分析的脚本`align_and_estimate_abundance.pl`的使用。

1. 命令
- `nohup align_and_estimate_abundance.pl --transcripts Trinity.fasta --seqType fq --left sample.clean_R1.fq.gz --right sample.clean_R2.fq.gz --est_method RSEM --aln_method bowtie2 --thread_count 12 --debug --prep_reference --trinity_mode --output_dir > bowtie.out 2>&1 &`
2. 必要参数：
- --transcripts: 指定 TRINITY 组装的转录本 fasta 文件，一般是Trinity.fasta。
- --seqType: 指定序列类型（fq或fa），fq 表示 fastq 格式。
- 指定测序reads：--samples_file指定文本文件，可包含多个样品多个生物学重复。单个样品的双端测序reads也可以指定--left 和 --right；单端测序reads指定--single。reads需要进行QC（FastQC,Trimmomatic或fastp等）之后再使用，定量分析结果更准确。
- --est_method: 选择用于估算丰度的方法(基于alignment的RSEM, eXpress(最新版本不支持)，不基于alignmetn的更快，有kallisto, salmon)，这里使用 RSEM。

--samples_file指定的文本文件格式如下(代表cond_A和cond_B两个样本，各包含三个生物学重复。如果是单端测序reads，第四列留空即可。)：

```
cond_A    cond_A_rep1    A_rep1_left.fq    A_rep1_right.fq
cond_A    cond_A_rep2    A_rep2_left.fq    A_rep2_right.fq
cond_A    cond_A_rep3    A_rep3_left.fq    A_rep3_right.fq
cond_B    cond_B_rep1    B_rep1_left.fq    B_rep1_right.fq
cond_B    cond_B_rep2    B_rep2_left.fq    B_rep2_right.fq
cond_B    cond_B_rep3    B_rep3_left.fq    B_rep3_right.fq
```

3. 可选参数 
- --aln_method: 选择比对工具，如 bowtie2或bowtie。
- --output_dir：指定输出文件保存的目录
- --thread_count：线程数，默认4。
- --debug：保留中间文件。
- --gene_trans_map <string>：包含基因和转录本关系的文件，每一行'gene(tab)transcript' identifiers，来生成基于基因的定量表达结果。
- --trinity_mode: 如果--transcripts是用trinity软件得到的转录本，可以用这个参数代替--gene_trans_map，会自动生成gene_trans_map需要的文件，并生成基于基因的定量表达结果。
- --prep_reference: 准备参考文件时需要使用的参数（构建目标的索引）。
- --SS_lib_type <string>：指定链特异性建库类型。paired('RF' or 'FR'), single('F' or 'R')。
- --samples_idx <int>：限制对样本条目的处理（索引从1开始）
- 另外还有一些调用软件（bowtie, bowtie2，RSEM，kallisto，salmon）用的参数也可以设定。

# 4. 结果文件
1. <sample>.genes.results：包含基因水平的表达估算结果。
- 主要列包括gene_id	transcript_id(s)	length	effective_length	expected_count  TPM	FPKM。
2. <sample>.isoforms.results：包含转录本水平的表达估算结果。
- 主要列包括transcript_id	gene_id	length	effective_length	expected_count	TPM   FPKM	IsoPct。
3. <sample>.isoforms.sorted.bam（可选的输出文件）：
- RSEM 对每个样本生成的比对后的 BAM 文件，包含转录本序列的比对信息。
- 这些 BAM 文件可以用于后续的可视化或其他分析。
4. <sample>.stat 文件夹
- 包括三个文件：<sample>.cnt,<sample>.model,<sample>.theta。
- 包含关于 RSEM 运行的一些统计信息和估算结果的总结。
- 包括比对率、有效比对质量等信息。
5. <sample>.transcript.bam（如果没有使用 --no-bam-output则生成）：
- 默认情况下生成的 BAM 文件，可用于直接查看与转录本的比对。
6. <sample>.temp 文件夹
- 如果使用--debug则会保留中间文件在这个文件夹。

生成的这些结果文件最常用于后续的差异表达分析、功能注释和生物学解释。

# 5. reference
1. https://github.com/trinityrnaseq/trinityrnaseq/wiki/Trinity-Transcript-Quantification

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" 