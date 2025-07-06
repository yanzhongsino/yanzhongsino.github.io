---
title: 从NCBI的SRA数据库下载原始测序数据
date: 2025-07-05
categories: 
- NCBI
- SRA
- download
tags:
- SRA
- Sequence Read Archive
- SRA Toolkit
- NGS
- TGS
- Illumina
- PacBio
description: 记录从NCBI的SRA数据库下载原始测序reads数据（二代数据和三代数据），并转换为fastq格式的方法。
---

<div align="middle"><iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/054NrUrqsXqh1WfZ9sbNdI?utm_source=generator" width="50%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe></div>

# 1. 下载原始测序数据
二代或三代测序的原始序列读数可以从NCBI的Sequence Read Archive(SRA)数据库下载：https://www.ncbi.nlm.nih.gov/sra。

# 2. 快速上手
从NCBI等公共数据库下载原始测序的raw reads数据，先下载SRA格式文件，然后转换为fastq格式文件。使用NCBI官方工具SRA Toolkit里的prefetch和fasterq-dump命令来下载和转换。

找到NCBI数据的SRR序列号，然后prefetch下载SRA格式数据，fasterq-dump转SRA格式为fastq格式。
- `prefetch SRR1234567 -p`
- `fasterq-dump --split-3 ./SRR1234567 -p -e 12`

# 3. SRA和SRA Toolkit简介
## 3.1. SRA
1. SRA格式
- SRA（Sequence Read Archive）格式是NCBI专用的储存高通量测序数据的格式，可以处理各种类型的测序技术生成的数据，包括Illumina、Ion Torrent、454、PacBio等。
2. SRA Normalized和SRA Lite格式的区别
- SRA Normalized是SRA的标准化格式，大部分情况下使用这个格式。
- SRA Lite是把SRA Normalized的质量分数（quality scores）进行简化，将碱基质量得分分为了pass和reject两种，pass统一给分为30，而reject统一给分为3。 
- 下载得到的SRA Normalized文件名为SRR1234567；下载得到的SRA Lite文件名为SRR1234567.lite.1（文件更小）。
- SRA转为fastq格式的操作对两种格式都有效。
## 3.2. SRA Toolkit
### 3.2.1. SRA Toolkit介绍
1. SRA Toolkit是由NCBI提供的一组工具，用于访问、下载和处理存储在SRA（Sequence Read Archive）中的高通量测序数据。这个工具套件允许用户从SRA数据库中检索数据、将数据格式转换为更常用的格式（如FASTQ）以及其他数据处理任务。
2. SRA Toolkit软件的使用manual： https://github.com/ncbi/sra-tools/wiki
### 3.2.2. SRA Toolkit下载和安装
1. conda安装：`conda install bioconda::sra-tools`
2. 手动安装
- 官方网站：https://github.com/ncbi/sra-tools/wiki/01.-Downloading-SRA-Toolkit
- 在网站上找到对应版本，复制链接；然后在linux系统用命令下载和解压缩即可使用
- 下载：`wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/3.2.1/sratoolkit.3.2.1-ubuntu64.tar.gz`；解压缩：`tar -vxzf sratoolkit.tar.gz`
- 命令在`PWD/sratoolkit.3.0.0-mac64/bin`目录下
3. 确认安装正确
- 查看命令：`which fastq-dump`
- 测试命令：`fastq-dump --stdout -X 2 SRR390728`；如果列出序列数据，则代表命令安装正确。
### 3.2.3. SRA Toolkit的命令
- SRA Toolkit包括下载SRA数据的命令`prefetch`,将SRA格式的数据转换为FASTQ格式的命令`fastq-dump`,`fasterq-dump`，检索SRA数据文件中的元数据的`vdb-dump`,将SRA文件直接转换为SAM格式（如果记录在数据库中有对映/SAM信息）的`sam-dump`。

# 4. 下载原始测序文件和转换格式
## 4.1. 查找SRA-accessions编号（SRR+6位数字组成的序列号）
1. 在NCBI网站 https://www.ncbi.nlm.nih.gov/sra 搜索物种名等信息，查询需要下载的原始测序数据，获取数据的SRA编号；
2. 可选择多个搜索结果，然后Send to-File-RunInfo下载SraRunInfo.csv表格文件，里面有非常完整的上传人填写的测序相关信息，包括SRA Lite格式的download_path；
3. 选择多个搜索结果，然后Send to-File-Acdession List下载SraAccList.csv表格文件。
4. 点击进入一个搜索结果，然后点击SRR号，再点击Data access栏，在SRA archive data栏就能看到两种格式的SRA数据的下载网址。

## 4.2. 下载SRA文件
1. prefetch调取下载
- 先在NCBI查询需要下载的数据的SRR编号，再使用编号信息下载SRA文件:`prefetch SRR1234567 SRR7654321`命令会下载SRA Normalized格式文件，支持多个SRR编号参数，依次下载。
- 多个SRA数据下载，也可以把SRR编号保存在文件中，一个编号一行（同SraAccList.csv表格文件）。然后用--option-file参数指定保存了编号的文件，来依次下载多个SRA数据：`prefetch --option-file SraAccList.csv`
- 为每个数据生成SRR编号命名的文件夹，里面只有一个以SRR1234567.sra命名的文件。
- prefetch默认单个文件最大20Gb，如果超过20Gb会被跳过，可以用`--max-size 100G`参数修改允许的最大文件大小。
- 在同一目录下多次运行，prefetch会自动检查已下载的文件，跳过下载好的文件。所以算是支持断点续传。
- 其他参数：
  - -X|--max-size: 设置最大下载文件大小，默认20G。
  - -p|--progress: 显示下载进度。
  - --eliminate-quals：下载SRA-Lite格式文件。

2. wget下载
- 复制下载网址之后用wget下载：`wget -c http:url`
- 下载完成后，如果是SRA Normalized格式，则生成SRR1234567文件；如果是SRA Lite格式，则生成SRR1234567.lite.1文件。

## 4.3. SRA格式转为fastq格式
SRA Toolkit有两个命令可以将SRA格式转为fastq格式：fastq-dump和fasterq-dump。

区别主要是：
- fastq-dump只能单线程运行，但支持-gzip和-bzip2参数输出压缩格式的fastq文件。
- fasterq-dump可以多线程运行，但不支持压缩格式的参数。

1. fastq-dump
- `fastq-dump --split-files SRR1234567`
- 参数：
  - --split-spot: 将双端测序分为两份,放在一个文件中。
  - --split-files: 将双端测序分为两份,放在两个文件，对于一方有而一方没有的reads直接丢弃。
  - --split-3 : 将双端测序分为两份,放在两个文件，对于一方有而一方没有的reads（unpaired reads）会单独放在一个文件夹里。
  - --gzip：进行gzip压缩，输出fastq.gz文件
  - --bz2：进行bzip2压缩，输出fastq.bz2文件
  - -X 1000：仅导出前1000条reads，用于测试。

2. fasterq-dump
- `fasterq-dump --split-3 ./SRR1234567 -p -e 12`
- 参数：
  - -s|--split-spot: 将双端测序分为两份,放在一个文件中。
  - -S|--split-files: 将双端测序分为两份,放在两个文件，对于一方有而一方没有的reads直接丢弃。
  - -3|--split-3 : 将双端测序分为两份,放在两个文件，对于一方有而一方没有的reads（unpaired reads）会单独放在一个文件夹里。
  - -p：显示进程。
  - -e 24：指定线程，默认6。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>