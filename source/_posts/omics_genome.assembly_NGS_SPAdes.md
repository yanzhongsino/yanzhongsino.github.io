---
title: 用SPAdes对Illumina二代测序数据进行基因组预组装
date: 2024-11-05
categories: 
- omics
- genome
- genome assembly
tags: 
- SPAdes
- Illumina
- NGS
- genome assembly
- contig
- scaffold
description: 在只有Illumina二代测序数据时，用SPAdes对Illumina二代测序数据进行基因组预组装，得到较长的contigs和scaffolds。
---  

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=117151&auto=1&height=32"></iframe></div>

# 1. SPAdes简介
SPAdes (St. Petersburg genome assembler) 是2012年首次开发的一个多功能工具包，用于组装和分析测序数据，并包括校错功能。 
- SPAdes 主要针对 Illumina 测序数据开发，但也可用于 IonTorrent。 大多数 SPAdes 管道都支持二代三代reads混合组装，即允许使用长读数（PacBio 和 Oxford Nanopore）作为补充数据。 
- SPAdes 软件包为分离株、单细胞细菌、元基因组和转录组数据的DNA组装提供了管道。
- 附加模式允许恢复细菌质粒和 RNA 病毒，以及执行 HMM 引导的组装。 
- 此外，SPAdes软件包还包括辅助工具，用于高效的k-mer计数和基于k-mer的读数过滤、组装图构建和简化、序列到图的比对以及元基因组分选细化。

最新的SPAdes v4.0.0版本支持NCBI的SRA文件作为输入。

# 2. SPAdes安装
1. 下载预安装的二进制文件

```shell
wget https://github.com/ablab/spades/releases/download/v4.0.0/SPAdes-4.0.0-Linux.tar.gz # 下载
tar -xzf SPAdes-4.0.0-Linux.tar.gz # 解压缩
./SPAdes-4.0.0-Linux/bin/spades.py -h # 查看参数
./SPAdes-4.0.0-Linux/bin/spades.py --test # 测试运行
```

2. conda安装：`conda install bioconda::spades`

# 3. SPAdes运行
1. 命令
- `nohup spades.py -o /path/to/output_directory/ -t 32 -1 /path/to/sample_clean_R1.fq.gz -2 /path/to/sample_clean_R2.fq.gz --careful 2>&1 > spades.log &`

2. 基本参数
- -o <output_dir>：指定输出文件目录
- -1, -2：指定paired-end测序文件
- --merged：指定合并的paired-end测序文件
- -s：指定single-end测序文件
- -t：指定运行线程
- -m int：设定内存的限制，单位为 Gb。如果程序使用的内存达到此值，则程序会终止运行。默认值是 250 。
- -k：kmer数，一次可以输入多个，用逗号分隔，数值从小到大排列，kmer最大为127，数值必须是奇数，一般自动选择即可，--sc参数，则默认值为 21,33,55。若没有 --sc参数，则程序会根据 reads长度自动选择 k-mer参数。
3. 运行模式参数
- --isolate：对于高覆盖率的分离和多细胞 Illumina 数据，强烈建议使用此参数；它能提高组装质量并缩短运行时间。 我们还建议在组装前对reads进行QC和trim。 该选项与--only-error-correction 或--careful 选项不兼容。
- --sc：对于MDA扩增单细胞(single-cell)数据使用此参数。假设覆盖范围极不均匀且存在扩增伪影。
- --meta：运行metaSPAdes模式，在组装元基因组数据集时使用此参数，同metaspades.py。 目前，metaSPAdes 只支持单个短读数文库，而且必须是成对的短读数文库（我们希望尽快取消这一限制）。 此外，您还可以提供长读数（例如使用--pacbio 或--nanopore 选项），但是元基因组的混合组装仍然是一个实验管道，无法保证最佳性能。 它不支持careful mode（不提供错配校正）。 此外，您不能为 metaSPAdes 指定覆盖率截止值。 请注意，metaSPAdes 可能对数据中残留的技术序列（比如接头序列）非常敏感，因此请对原始数据进行质量控制并对数据进行相应的预处理后再使用。
- --plasmid：运行plasmidSPAdes模式，在用WGS数据集组装质粒时用此参数，同plasmidspades.py。与单细胞模式(--sc)不兼容。不推荐在一个以上的库中运行此模式。
- --metaplasmid和--metaviral：这些参数专门用于从元基因组组装中提取染色体外的元素。 它们运行类似的管道，但在简化步骤上略有不同；一个区别是，在metaviral模式下，我们输出线性的推断的染色体外contigs，而在metaplasmid模式下，我们不输出线性的推断的染色体外contigs。 此外，对于plasmidSPAdes、metaplasmidSPAdes和metaviralSPAdes，我们建议使用viralVerify工具验证生成的contigs。
- --bio：生物合成模式。
- --rna：RNA-seq数据模式。 
- --rnaviral：病毒RNA数据模式。
- --corona：启用HMM引导的冠状病毒组装模块。
- --iontorrent：IonTorrent数据模式，允许BAM格式文件作为输入。
- --sevage：SARS-CoV-2废水样本模式。
4. 管道运行参数
- --careful：通过运行 MismatchCorrector模块进行基因组上 mismatches和 short indels的修正。推荐使用此参数。
- --only-error-correction：仅仅执行 reads error correction 步骤
- --only-assembler：仅仅运行组装模块
- --continue：从上一次终止处继续运行程序。
- --restart-from <check_point>：从指定的位置重新开始运行程序。和上一个参数相比，此参数可以用于改变一些组装参数。可选的值有：ec 从 error correction 处开始；as 从 assembly module 处开始；k<int> 从指定的 k 值处开始；mc 从 mismatch correction 处开始；last从最后可用的check-point开始，类似--continue。
- --checkpoints <mode>：生成一些checkpoints，允许SPAdes从运行中间阶段重新开始运行。可用的mode包括none（默认），all（生成所有checkpoints），last（只在SPAdes报错时生成一个checkpoint）。
5. 其他参数
- --phred-offset：碱基质量格式， 33 或 64。默认自动检测。
6. 运行时间：32个线程，34Gb Illumina paired-end clean reads，9：40-

# 4. SPAdes输出结果
1. 结果文件
- <output_dir>/corrected/：directory contains reads corrected by BayesHammer in *.fastq.gz files; if compression is disabled, reads are stored in uncompressed *.fastq files
- <output_dir>/scaffolds.fasta：即组装结果scaffolds文件，推荐作为结果文件或下一步分析使用。
- <output_dir>/contigs.fasta：即组装结果contigs文件
- <output_dir>/assembly_graph_with_scaffolds.gfa：contains SPAdes assembly graph and scaffolds paths in GFA 1.2 format
- <output_dir>/assembly_graph.fastg：contains SPAdes assembly graph in FASTG format
- <output_dir>/contigs.paths：contains paths in the assembly graph corresponding to contigs.fasta
- <output_dir>/scaffolds.paths：contains paths in the assembly graph corresponding to scaffolds.fasta


# 5. reference
1. github：https://github.com/ablab/spades
2. 官方教程：http://ablab.github.io/spades/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>