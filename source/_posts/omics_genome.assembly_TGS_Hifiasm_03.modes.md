---
title: 用Hifiasm组装基因组：（三）Hifiasm软件组装基因组的多种模式
date: 2024-06-21
categories: 
- omics
- genome
- genome assembly
tags:
- genome assembly
- third generation sequencing
- TGS
- Hifiasm
- HiFi
- PacBio
- Hi-C
- ONT

description: 介绍基于三代HiFi reads用软件Hifiasm进行基因组的从头组装的多种模式，以及每种模式的命令。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2153506013&auto=1&height=32"></iframe></div>


# 1. Hifiasm组装基因组的模式
根据可用的数据不同，Hifiasm组装基因组在HiFi数据基础上，还有几种模式可以增加组装的完整度和准确度。

1. HiFi-only assembly 模式（只有HiFi数据）
2. Trio-binning模式（HiFi数据+父母本二代Illumina测序数据）
3. Hi-C Integrated assembly 模式（HiFi数据+Hi-C数据）
4. 端对端组装：HiFi+ONT模式（HiFi数据+ONT超长reads数据）

# 2. HiFi-only assembly 模式（只有HiFi数据）
## 2.1. 经典模式
1. 命令
- `nohup hifiasm -o sample_prefix -t 32 Hifi.fq.gz 2>&1 > hifiasm.log &`
2. 参数
- HiFi reads可以是fq或fa格式（fq的质量值会被忽略），可以是gz压缩格式。
- -o指定输出文件前缀；-t指定线程。
- 用命令`2>&1 >hifiasm.log`保存日志和报错内容到hifiasm.log文件。

## 2.2. 两种组装方式
1. 单倍体分型组装（two partially phased assembly）
- 默认是以此方式组装。
- 单倍体分型组装生成一对文件（asm.bp.hap1.p_ctg.gfa和asm.bp.hap2.p_ctg.gfa），代表二倍体的两个单倍型。同时也会生成primary contigs文件asm.bp.p_ctg.gfa。
2. primary/alternate组装
- 加一个参数`--primary`则指定primary/alternate组装方式。
- 命令：`nohup hifiasm -o sample_prefix -t 32 --primary Hifi.fq.gz 2>&1 > hifiasm.log &`
- 分别生成primary contigs和alternate contigs文件asm.p_ctg.gfa和asm._ctg.gfa。

# 3. Trio-binning模式（HiFi数据+父母本二代Illumina测序数据）
1. 介绍
- 当父母本的二代Illumina reads可用时，也可以通过trio binning生成一对解析的单倍型的组装。
- Hifiasm中用到的trio binning 技术是指利用父本、母本和子代的遗传信息对子代的单倍型划分的方法。该方法的有效性随着杂合度的增加而提高，极大地提升了等位基因组的组装质量。
2. 命令

```shell
# trio-binning模式需要额外安装yak，两种安装方式任选一种
# source code
git clone https://github.com/lh3/yak
cd yak && make
# bioncda
conda install -c bioconda yak

# 运行组装
yak count -b37 -t16 -o pat.yak <(cat paternal_1.fq.gz paternal_2.fq.gz) <(cat paternal_1.fq.gz paternal_2.fq.gz)
yak count -b37 -t16 -o mat.yak <(cat maternal_1.fq.gz maternal_2.fq.gz) <(cat maternal_1.fq.gz maternal_2.fq.gz)
hifiasm -o sample_prefix -t 32 -1 pat.yak -2 mat.yak Hifi.fq.gz 2>&1 > hifiasm.log &
```

3. 参数解释
- 命令中Illumina双端测序的父本paternal数据和母本maternal数据同时使用

# 4. Hi-C Integrated assembly 模式（HiFi数据+Hi-C数据）
1. 介绍
- 当Hi-C数据可用时，可以生成一对解析的单倍型的组装。
- 李恒团队2022年在Nature biotechnology上发表论文Haplotype-resolved assembly of diploid genomes without parental data（https://www.nature.com/articles/s41587-022-01261-x），在Hifiasm中引入了Hi-C Integrated assembly 模式。
- Hi-C Integrated assembly模式针对PacBio HiFi (High-Fidelity) 长读长测序技术和Hi-C (High-Throughput Chromatin Confirmation Capture) 测序技术进行了全新的设计。
- 该算法结合了HiFi数据中精确的局部单倍型信息和Hi-C数据中的长距离互作用信息以达到全局定相 (phasing)，从而获得不依赖亲本信息的染色体级别的单倍型组装结果。为了进一步提高组装质量，作者充分利用了组装图中的结构信息，以及其前期研究中的Graph-binning等策略。
- 这个模式组装后的基因组还未挂载在染色体上，仍然需要Juicer+3ddna+juicebox等软件进行染色体挂载。
- 这个模式的数据最易获得，所以也很常用。

2. 命令
- `nohup hifiasm -o sample_prefix -t 32  --h1 HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fq.gz 2>&1 > hifiasm.log &`
3. 参数
- 用--h1和--h2指定Hi-C数据。

# 5. 端对端组装：HiFi+ONT模式（HiFi数据+ONT超长reads数据）
当ONT数据可用时，可以集成超长ONT数据生成端粒到端粒的组装
1. 命令
- `nohup hifiasm -o sample_prefix -t 32 --ul ONT.fq.gz Hifi.fq.gz 2>&1 > hifiasm.log &`
2. 参数
- 用--ul指定ONT数据。


# 6. references
1. Hifiasm manual：https://hifiasm.readthedocs.io/_/downloads/en/latest/pdf/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>