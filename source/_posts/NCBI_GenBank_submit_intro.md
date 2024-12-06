---
title: 提交核苷酸序列到NCBI的GenBank的介绍
date: 2024-12-06
categories: 
- NCBI
- GenBank
- submit
tags:
- SRA
- Unassembled sequence reads
- submit
- NCBI
- NLM
- GenBank
- NGS
- TGS
- WGS
- TSA
- TPA
- TLS
- BankIt
- table2asn
description: 介绍NCBI,GenBank和向GenBank上传核苷酸序列。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1392871583&auto=1&height=32"></iframe></div>

# 1. NCBI和GenBank介绍
1. NCBI
- NCBI：National Center for Biotechnology Information，美国国家生物技术信息中心。它是美国国家卫生机构 (National Institutes of Health, NIH) 的分支机构 国家医药馆 (National Library of Medicine, NLM) 的一部分。
- NCBI 拥有一系列与生物技术和生物医学相关的数据库，是生物信息学工具和服务的重要资源。可以上传、下载、检索、学习、分析和研究生物和医药相关数据，并支持开发API和代码库。
- NCBI由大卫-利普曼（David Lipman）领导，他是BLAST序列比对程序的原作者之一，也是生物信息学领域广受尊敬的人物。
- NCBI的网站是：https://www.ncbi.nlm.nih.gov/。
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

# 2. 向GenBank上传序列
## 2.1. 上传序列的类型
https://www.ncbi.nlm.nih.gov/genbank/submit_types/ 提供了向GenBank上传序列的类型说明。

GenBank只接受由上传者直接生成的转录组或基因组序列，通常要求包括生物源信息和序列的注释。

下面两种情况应选择NCBI的其他数据库：
- 二代或三代测序平台的原始序列读数应提交至Sequence Read Archive(SRA)：https://www.ncbi.nlm.nih.gov/sra
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

## 2.2. 上传序列的工具
1. BankIt 【推荐】
- 基于WWW网络的提交工具，有向导指引提交过程。
- BankIt网址：https://www.ncbi.nlm.nih.gov/WebSub/
2. Submission Portal
- Submission Portal网址：https://submit.ncbi.nlm.nih.gov/
- Submission Portal是一个适用于多种数据类型的提交门户系统。
- 目前，核糖体 RNA (rRNA)、rRNA-ITS、metazoan 线粒体 COX1、真核细胞核 mRNA、流感、诺如病毒、登革热或 SARS-CoV-2 序列只可以通过Submission Portal的 GenBank 组件提交。
- 这个工具今后还将扩展到其他类型的 GenBank 提交。
3. Sequin：旧版提交工具 【不推荐】
- Sequin是以前常被使用的旧版GenBank提交工具，在2021.01退役，原来的界面也404了：https://www.ncbi.nlm.nih.gov/Sequin。
- Sequin 是 NCBI 开发的一款独立软件工具，用于向 GenBank、EMBL 和 DDBJ 数据库提交和更新序列。 Sequin 能够处理长序列和序列集（分段条目以及种群、系统发育和突变研究）。 它还允许序列编辑和更新，并提供复杂的注释功能。 此外，Sequin 还包含许多内置的验证功能，以加强质量保证。
4. table2asn：一个提交序列的准备工具
- 它主要用于提交注释的基因组和大批量序列的前期准备，把注释的table格式文件转为asn文件。
- 它会生成不符合GenBank要求的注释说明，所以还可以用于基因组注释错误的检测。
- 可以查看博客【提交基因组到公共数据库】：https://yanzhongsino.github.io/2022/03/22/NCBI_GenBank_submit_genome/。里面记录了table2asn的具体用法。
- table2asn网址：https://www.ncbi.nlm.nih.gov/genbank/table2asn/
- table2asn是一个命令行程序，可取代旧版工具 tbl2asn。

# 3. 上传序列后
1. 收到提交的序列后，GenBank 工作人员会检查数据的原创性，为序列分配ID（accession number），并进行质量检查。
2. 然后，提交的序列将被发布到公共数据库中，这些条目可通过 Entrez 检索或通过 FTP 下载。
3. 在学术论文或其他地方，可以提供accession number来引用生物序列数据。

# 4. 更新或修订 GenBank 序列
1. 提交者可随时对 GenBank 条目进行修订或更新。 
2. 关于不同类型更新的正确格式，请参阅更新指南页面:https://www.ncbi.nlm.nih.gov/genbank/update/。
3. 将更新和修订发送至 gb-admin@ncbi.nlm.nih.gov。请务必在邮件主题行注明要更新序列的登录号。

# 5. 用BankIt的上传步骤

BankIt：https://www.ncbi.nlm.nih.gov/WebSub

建议先注册和登录NCBI账号，之后的所有提交记录都会保存在你的账号下，在这里可以查看提交记录：https://submit.ncbi.nlm.nih.gov/subs/。

## 5.1. 选择上传的数据类别
下面列几个常用的：
- Eukaryotic and Prokaryotic Genomes(WGS or Complete): 组装好的真核和原核物种的基因组，可以是未注释的。
- Transcriptome Shotgun Assembly (TSA)：组装好的转录组
- Unassembled sequence reads (SRA)：未组装的测序reads
- Sequence data not listed above：mRNA, genomic DNA, organelle, ncRNA, plasmids...：其他测序数据，细胞器基因组选这个。

不同的数据类型，上传步骤需要填写的内容不同。

## 5.2. 以上传SRA数据为例，大概说明一下BankIt上传数据的步骤和要填的内容
1. 提交者信息Submitter
- 包括姓名First and Last name，邮箱Email，单位Submitting organization(学校)，部门Department(学院)，街道Street，城市City，省/州State/Province，邮编Postal code，国家Country。
- 在最后可以选择**Update my contact information in profile**会更新填写的信息到账号，下一次新填写就默认填入提交者信息了。
2. General info
3. Project info
4. BioSample Type
5. BioSample Attributes
6. SRA Metadata
7. Files：上传文件。
8. Review&Submit：最后检查一遍信息没错误就确认提交。


没什么问题的话，两个工作日内会发邮件告知GenBank accession numbers，可用于文章引用。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>