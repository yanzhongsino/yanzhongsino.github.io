---
title: 使用BankIt提交生物数据到NCBI的GenBank
date: 2024-12-06
categories: 
- NCBI
- GenBank
- submit
tags:
- SRA
- Unassembled sequence reads
- submit
- GenBank
- NGS
- TGS
description: 上传生物数据到NCBI的GenBank数据库的操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=509092165&auto=1&height=32"></iframe></div>

# NCBI和GenBank介绍
1. NCBI
- NCBI：National Center for Biotechnology Information，美国国家生物技术信息中心。它是美国国家卫生机构 (National Institutes of Health, NIH) 的分支机构 国家医药馆 (National Library of Medicine, NLM) 的一部分。
- NCBI 拥有一系列与生物技术和生物医学相关的数据库，是生物信息学工具和服务的重要资源。可以上传、下载、检索、学习、分析和研究生物和医药相关数据，并支持开发API和代码库。
- NCBI由大卫-利普曼（David Lipman）领导，他是BLAST序列比对程序的原作者之一，也是生物信息学领域广受尊敬的人物。
- NCIBI的网站是：https://www.ncbi.nlm.nih.gov/。
2. NCBI 的数据库
- DNA 序列数据库 **GenBank** 
- 生物医学文献数据库 **PubMed**
- NCBI 表观基因组学数据库 **NCBI Epigenomics database**
- 在线人类孟德尔遗传数据库 **Online Mendelian Inheritance in Man**
- 分子建模数据库 （3D蛋白结构）**Molecular Modeling Database**
- 单核苷酸多态性数据库 **dbSNP**
- 参考序列数据集 **Reference Sequence Collection**
- 人类基因组图谱 **a map of the human genome,**
- 生物分类浏览器 **a taxonomy browser**
- 与美国国家癌症研究所合作提供 癌症基因组解剖项目 **Cancer Genome Anatomy Project**

所有这些数据库都可通过 Entrez 搜索引擎在线获取。 

3. GenBank
- GenBank是一个开放存取的注释数据库，包含所有公开的核苷酸序列及其蛋白质翻译。1979年被开发出来。
- GenBank 接受原始生物序列的上传和下载，目前是全世界生物和医学领域最完整和最常用的核苷酸序列数据库。
- GenBank的网址是：https://www.ncbi.nlm.nih.gov/genbank

# 向GenBank上传序列
## 上传序列的类型
https://www.ncbi.nlm.nih.gov/genbank/submit_types/ 提供了向GenBank上传序列的类型说明。

GenBank只接受由上传者直接生成的转录组或基因组序列，通常要求包括生物源信息和序列的注释。

下面两种情况应选择NCBI的其他数据库：
- 二代测序平台的原始序列读数应提交至Sequence Read Archive(SRA)：https://www.ncbi.nlm.nih.gov/sra
- 非由提交者直接获得的序列数据应提交到Third Party Annotation database （第三方注释数据库）：https://www.ncbi.nlm.nih.gov/genbank/tpa

1. 标准上传：mRNA 或基因组序列
- 提交的数据必须包括源生物信息和提交者提供的注释。
- 必须是由提交者直接生成的数据。
2. 完整的微生物基因组（Complete Microbial Genomes）
- 提交指南 Bacterial Genome Submission Guidelines：https://www.ncbi.nlm.nih.gov/genbank/genomesubmit/
3. 全基因组散弹枪序列 （Whole Genome Shotgun (WGS) Sequences）
- 可提交原核生物和真核生物的全基因组组装序列或组装的草稿基因组contigs序列，可以不提供基因组注释。
- WGS提交指南：https://www.ncbi.nlm.nih.gov/genbank/wgs/
4. 转录组散弹枪序列（Transcriptome Shotgun Assembly (TSA) Sequences）
- 可以提交组装好的转录组或组装的草稿转录组contigs序列，组装用的数据源可以是提交到SRA数据库的原始测序数据。
- TSA提交指南：https://www.ncbi.nlm.nih.gov/genbank/tsa/
5. 高通量基因组序列（High-Throughput Genomic (HTGs) Sequences）
- 基于克隆的高通量基因组序列，通常是cosmids或BACs。
- HTGs提交指南：https://www.ncbi.nlm.nih.gov/genbank/htgs/
6. 第三方注释（Third Party Annotation (TPA)）
- TPA数据库接受基因组序列的注释，或者派生的/组装的序列的第三方注释信息。
- 提交的 TPA 必须包括 GenBank 中已有的序列数据，注释所依据的分析必须发表在同行评审的科学杂志上。
- TPA 提交指南：https://www.ncbi.nlm.nih.gov/genbank/tpa
7. 靶向基因位点研究（Targeted Locus Study (TLS)）
- 靶向基因位点研究（TLS）是一项大规模的靶向测序项目（>2,500 个序列），研究对象可以是来自多个生物体的单个基因座，也可以是来自单个生物体的多个保守元素。 
- TLS 提交指南：https://www.ncbi.nlm.nih.gov/genbank/tlsguide/
8. GenBank 不接受以下数据：
- 非连续序列
- 引物序列
- 无核苷酸的蛋白质序列
- 同时含有基因组和 mRNA 的混合序列
- 无实体对应物的序列（一致性序列，consensus sequences）
- 长度小于 200 个核苷酸的序列

## 上传序列的工具

1. BankIt 【推荐】
- 基于WWW的提交工具，有向导指引提交过程。


2. Submission Portal：
- 一个适用于多种数据类型的提交门户系统。
- 目前，只有核糖体 RNA (rRNA)、rRNA-ITS、metazoan 线粒体 COX1、真核细胞核 mRNA、流感、诺如病毒、登革热或 SARS-CoV-2 序列可以通过该工具的 GenBank 组件提交。
- 基因组和转录组数据可以分别通过基因组门户网站和 TSA 门户网站提交。
- 这个工具今后还将扩展到其他类型的 GenBank 提交。
3. table2asn
- table2asn：https://www.ncbi.nlm.nih.gov/genbank/table2asn/
- table2asn是一个命令行程序，可取代旧版工具 tbl2asn。
- 用于自动创建序列记录，为提交序列到 GenBank 做前期准备。
- 它主要用于提交注释的基因组和大批量序列，把注释的table格式文件转为asn文件，可通过 FTP 在 MAC、PC 和 Unix 平台上使用。

更新或修订 GenBank 序列
提交者可随时对 GenBank 条目进行修订或更新。 关于不同类型更新的正确格式，请参阅更新指南页面。 将更新和修订发送至 gb-admin@ncbi.nlm.nih.gov 。 请务必在主题行注明要更新序列的登录号。


只能向 GenBank 提交原始序列。 直接向 GenBank 提交序列可使用 BankIt（一种基于网络的表格）或独立提交程序 Sequin。 收到提交的序列后，GenBank 工作人员会检查数据的原创性，为序列分配登录号，并进行质量保证检查。 然后，提交的序列将被发布到公共数据库中，这些条目可通过 Entrez 检索或通过 FTP 下载。 表达序列标签（EST）、序列标记位点（STS）、基因组调查序列（GSS）和高通量基因组序列（HTGS）数据的批量提交通常由大型测序中心提交。 GenBank 直接提交组也处理完整的微生物基因组序列。

# 上传步骤
## 上传工具
BankIt：https://www.ncbi.nlm.nih.gov/WebSub/index.cgi

使用BankIt在线上传，允许一次提交多个细胞器基因组序列。

## 上传步骤
1. 选择上传的数据类别。细胞器基因组选择Sequence data not listed above：organelle。下面列几个常用的：
- Eukaryotic and Prokaryotic Genomes(WGS or Complete): 组装好的真核和原核物种的基因组
- Transcriptome Shotgun Assembly (TSA)：组装好的转录组
- Unassembled sequence reads (SRA)：未组装的测序reads
- Sequence data not listed above：mRNA, genomic DNA, organelle, ncRNA, plasmids...：其他测序数据，细胞器基因组选这个。
2. Contact：填写上传人的信息，包括姓名，学院，学校，地址，城市，地区/省份，邮编，国家，和接收上传信息的邮箱【重要】。
3. Reference：填写提供序列的作者和出版信息。
4. Sequencing Technology：填写测序方法信息，包括测序平台，是否是组装的数据，组装软件和版本，组装样品名称，覆盖度。
5. Nucleotide：填写序列的信息。
- 序列发布的时间，可以指定日期，也可以一通过上传审核就发布。
- 分子类型（Molecule Type）：细胞器基因组选的genomic DNA
- 拓扑结构（Topology）：线型分子（Linear）还是环形分子（Circular）。
- 是否是完整的细胞器基因组：yes/no。
- 核苷酸序列格式：fasta或者alignment，选的fasta
- 上传细胞器基因组的fasta文件
6. Organism：填写Organism name信息，可填物种的学名。
7. Submission Category：测序reads是上传者测序的还是使用的其他已上传序列数据。如果是使用其他已上传reads进行的组装则需要提供已上传reads的accession number。
8. Source Modifiers：资源信息。
- Organelle/Location: 叶绿体/线粒体/其他器官。
- Source Modifier可以填写Country；对应的value填写China。
9.  Features：提供注释信息，可以选择tbl文件或者手动填写注释表格，tbl文件的ID和提交的序列ID需要一致。
10. Review and Correct：回顾和确认填写的信息，即可完成提交。

没什么问题的话，两个工作日内会发邮件告知GenBank accession numbers，可用于文章引用。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>