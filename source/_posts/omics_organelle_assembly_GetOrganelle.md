---
title: 用GetOrganelle组装细胞器基因组或核基因组的核糖体RNA序列
date: 2024-12-12
categories:
- omics
- organelle
- assembly
tags:
- organelle
- mitogenome
- mitochondria
- plastome
- chloroplast
- assembly
- fastg
- WGS
- nuclear ribosomal RNA
- 5.8S
- 18S
- 26S
- ITS

description: 介绍GetOrganelle，使用GetOrganelle基于whole genome skimming data组装植物质体、植物线粒体、动物线粒体、真菌线粒体、核基因组的核糖体RNA（18S-ITS1-5.8S-ITS2-26S）等序列。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1312166694&auto=1&height=32"></iframe></div>

# 1. GetOrganelle简介
GetOrganelle是开发用Illumina reads（双端测序/单端测序）来组装细胞器基因组的软件，可以组装植物质体、植物线粒体、动物线粒体、真菌线粒体等细胞器基因组。

# 2. GetOrganelle 工作流
<img src="https://user-images.githubusercontent.com/8598031/83836465-85afa080-a6c1-11ea-8b08-b08623d974f4.png" title="GetOrganelle flowchart" width="60%"/>

**<p align="center">Figure 1. GetOrganelle flowchart 图源：https://github.com/Kinggerm/GetOrganelle</p>**

# 3. 安装
1. conda 安装：`conda install bioconda::getorganelle`

# 4. 下载和配置数据库
1. 下载和配置高等植物质体和线粒体基因组库：`get_organelle_config.py --add embplant_pt`
2. 所有库
- get_organelle_config.py --add embplant_pt       #配置高等植物质体基因组库
- get_organelle_config.py --add embplant_mt      #配置高等植物线粒体基因组库
- get_organelle_config.py --add other_pt              #配置其他植物质体基因组库
- get_organelle_config.py --add fungus_mt           #配置真菌线粒体基因组库
- get_organelle_config.py --add animal_mt           #配置动物线粒体基因组库
- get_organelle_config.py --add embplant_nr       #配置高等植物核糖体DNA库
- get_organelle_config.py --add fungus_nr            #配置真菌核糖体DNA库

# 5. 测试安装的运行
1. 下载一个模拟的拟南芥WGS数据
- `wget https://github.com/Kinggerm/GetOrganelleGallery/raw/master/Test/reads/Arabidopsis_simulated.1.fq.gz`
- `wget https://github.com/Kinggerm/GetOrganelleGallery/raw/master/Test/reads/Arabidopsis_simulated.2.fq.gz`
2. 检查数据完整性
- `md5sum Arabidopsis_simulated.*.fq.gz`
- `# 935589bc609397f1bfc9c40f571f0f19  Arabidopsis_simulated.1.fq.gz`
- `# d0f62eed78d2d2c6bed5f5aeaf4a2c11  Arabidopsis_simulated.2.fq.gz`
3. 组装质体
- `get_organelle_from_reads.py -1 Arabidopsis_simulated.1.fq.gz -2 Arabidopsis_simulated.2.fq.gz -t 1 -o Arabidopsis_simulated.plastome -F embplant_pt -R 10`
4. 结果
- 得到的结果文件同这个网址一致：https://github.com/Kinggerm/GetOrganelleGallery/tree/master/Test/results/Arabidopsis_simulated.plastome

# 6. 组装细胞器基因组
## 6.1. 输入数据-reads
1. 双端测序和单端测序的Illumina reads都支持。
2. 开发者建议使用未进行质控但去除了接头序列(adapter-trimmed)的raw reads进行组装。
3. 大部分被子植物的质体只需要 > 1G per end 的数据量，大部分线粒体只需要 > 5G per end 的数据量。

## 6.2. 参数
参考 https://github.com/Kinggerm/GetOrganelle/wiki/FAQ 微调参数。

1. 常用参数
- -1 FQ_FILE_1          Input file with forward paired-end reads (*.fq/.gz/.tar.gz).
- -2 FQ_FILE_2          Input file with reverse paired-end reads (*.fq/.gz/.tar.gz).
- -u UNPAIRED_FQ_FILES  Input file(s) with unpaired (single-end) reads.
- -o OUTPUT_BASE        Output directory.
- -s SEED_FILE          Input fasta format file as initial seed. Default:/home/zhongyan/.GetOrganelle/SeedDatabase/*.fasta 用相近物种的完整细胞器基因组序列作为种子有助于组装（如降解的DNA样本，动物/真菌的快速进化样本等）。
- -w WORD_SIZE          Word size (W) for extension. Default: auto-estimated  虽然自动估计的word size不能确保最佳性能或最佳结果，但如果生成的是完整/圆形细胞器基因组组装，则无需调整此参数 (-w)，因为 GetOrganelle 生成的圆形结果在不同参数和种子下高度一致。在某些动物有丝分裂基因组数据中，由于覆盖率估计不准确，自动估计的word size可能会出现问题，这时您需要对其进行微调。
- -R MAX_ROUNDS         Maximum extension rounds (suggested: >=2). Default: 15 (embplant_pt)
- -F ORGANELLE_TYPE     Target organelle genome type(s): embplant_pt/other_pt/embplant_mt/embplant_nr/animal_mt/fungus_mt/fungus_nr/anonym/embplant_pt,embplant_mt/other_pt,embplant_mt,fungus_mt
- --max-reads：在每个文件中使用reads的最大数量. Default:1.5E7 (-F embplant_pt/embplant_nr/fungus_mt/fungus_nr); 7.5E7 (-F embplant_mt/other_pt/anonym); 3E8 (-F animal_mt)。可以设置`--max-reads inf`来尽量使用文件中所有reads。
- --fast FAST_STRATEGY  ="-R 10 -t 4 -J 5 -M 7 --max-n-words 3E7 --larger-auto-ws --disentangle-time-limit 360"
- -k SPADES_KMER        SPAdes kmer settings. Default: 21,55,85,115 最佳kmer值。越多kmer值耗时越长。但建议使用范围宽的kmer，至少包含21和85。
- -t THREADS：使用的最大线程数。Default: 1
- -P PRE_GROUPED        Pre-grouping value. Default: 200000
- -v, --version         show program's version number and exit
- -h：查看常用参数的简短介绍。
- --help：查看所有参数的详细介绍。
2. 参数的常用用法
- `--max-reads inf --reduce-reads-for-coverage inf`：使用文件中所有reads。如果细胞器基因组上有许多异质性，则会有很多低频reads需要被用于组装，这时使用所有reads会帮助增加组装的完整性。

## 6.3. 运行组装
1. 组装有胚植物质体基因组：`get_organelle_from_reads.py -1 forward.fq -2 reverse.fq -o plastome_output -R 15 -k 21,45,65,85,105 -F embplant_pt`
2. 组装有胚植物线粒体基因组
- `get_organelle_from_reads.py -1 forward.fq -2 reverse.fq -o mitochondria_output -R 20 -k 21,45,65,85,105 -P 1000000 -F embplant_mt`
- (1) 使用 FASTG 文件用于下游手动调整的输出文件。目前植物线粒体基因组的大量重复序列很容易使得输出的FASTA格式有问题。
- (2) 由于植物线粒体基因组的复杂性，GetOrganelle论文中没有测试 embplant_mt 模式。
3. 组装有胚植物的核糖体RNA（18S-ITS1-5.8S-ITS2-26S）
- `get_organelle_from_reads.py -1 forward.fq -2 reverse.fq -o nr_output -R 10 -k 35,85,115 -F embplant_nr`
- 参考指引 FAQ：https://github.com/Kinggerm/GetOrganelle/wiki/FAQ#why-does-getorganelle-generate-a-circular-genome-or-not-for-embplant_nrfungus_nr`
4. 非有胚生物的细胞器基因组可能与有胚生物的不同
- 有一个内置的 other_pt 模式和为非有胚生物质体准备的默认数据库。
5. 组装真菌线粒体基因组
- `get_organelle_from_reads.py -1 forward.fq -2 reverse.fq -R 10 -k 21,45,65,85,105 -F fungus_mt -o fungus_mt_out`
6. 组装真菌核糖体RNA（18S-ITS1-5.8S-ITS2-28S）
- `get_organelle_from_reads.py -1 forward.fq -2 reverse.fq -R 10 -k 21,45,65,85,105 -F fungus_nr -o fungus_nr_out`
7. 组装动物线粒体基因组
- `get_organelle_from_reads.py -1 forward.fq -2 reverse.fq -R 10 -k 21,45,65,85,105 -F animal_mt -o animal_mt_out`

## 6.4. 结果
1. 主要结果文件
- *.path_sequence.fasta, each fasta file represents one type of genome structure
- *.selected_graph.gfa, the organelle-only assembly graph
- get_org.log.txt, the log file
- extended_K*.assembly_graph.fastg, the raw assembly graph
- extended_K*.assembly_graph.fastg.extend_embplant_pt-embplant_mt.fastg, a simplified assembly graph
- extended_K*.assembly_graph.fastg.extend_embplant_pt-embplant_mt.csv, a tab-format contig label file for bandage visualization
2. 结果的处理
- 如果生成的基因组是完整的（在日志文件和 \*.fasta 名称中显示），可以删除上述以外的其他文件。 
- 如果 GetOrganelle 无法生成完整的环状基因组（生成 \*scaffolds\*path_sequence.fasta），可以按照：https://github.com/Kinggerm/GetOrganelle/wiki/FAQ#what-should-i-do-with-incomplete-resultbroken-assembly-graph 中Output部分的指引调整参数进行第二次运行。
- 实在无法组装出完整的细胞器基因组，也可以使用不完整序列进行下游分析。

# 7. 用Assembly Graph组装
1. 用Assembly Graph组装的情况
- 通常我们使用raw reads进行组装，但有些复杂的细胞器基因组（如植物线粒体基因组）可能不能通过一次组装得到完整的基因组。这时，可以使用组装图（assembly graph）来组装。
- 组装图通常是\*.fastg格式或gfa格式文件，可以从之前的组装中获取，比如使用Bandage手动调整后导出的fastg文件或gfa文件，或者从三代测序reads中获取。
2. 从组装图中提取质粒基因组的命令
- `get_organelle_from_assembly.py -F embplant_pt -g ONT_assembly_graph.gfa`

# 8. references
1. github：https://github.com/Kinggerm/GetOrganelle
2. paper：https://genomebiology.biomedcentral.com/articles/10.1186/s13059-020-02154-5

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>