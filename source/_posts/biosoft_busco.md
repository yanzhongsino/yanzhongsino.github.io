---
title: busco的安装与使用
date: 2021-07-24 11:20:00
categories: 
- bio
- biosoftware
tags: 
- genome assessment
description: 记录评估基因组组装和注释完整性的工具busco的安装使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=3986241&auto=1&height=32"></iframe></div>

# busco简介
Benchmarking Universal Single-Copy Orthologs (BUSCO)是用于评估基因组组装和注释的完整性的工具。通过与已有单拷贝直系同源数据库的比较，得到有多少比例的数据库能够有比对，比例越高代表基因组完整度越好。

可以评估三种数据类型（1.组装的基因组；2.转录组；3.注释到的基因对应的氨基酸序列），使用需要评估的生物类别所属的数据库（从busco数据库下载）比对，得出比对上数据库的完整性比例的信息。

[busco官网](https://busco.ezlab.org)
[busco v5数据库](https://busco-data.ezlab.org/v5/data/lineages/)

# busco安装
`conda install -c bioconda busco=5.1.3` #安装版本是5.1.3

# busco数据库下载
`busco --list-datasets` #查看busco可用的数据库。

下载对应的busco数据库(wget + 数据库网址）；目前有v1-v5版本，根据需要评估的物种，尽量选用最新版本的最多基因的数据库。

植物相关的数据库有：

|类群|数据库|BUSCO groups数量|
|---|---|---|
|真核生物|[eukaryota_odb10.2020-09-10.tar.gz](https://busco-data.ezlab.org/v5/data/lineages/eukaryota_odb10.2020-09-10.tar.gz)|255|
|绿色植物|[viridiplantae_odb10.2020-09-10.tar.gz](https://busco-data.ezlab.org/v5/data/lineages/viridiplantae_odb10.2020-09-10.tar.gz)|425|
|有胚植物|[embryophyta_odb10.2020-09-10.tar.gz](https://busco-data.ezlab.org/v5/data/lineages/embryophyta_odb10.2020-09-10.tar.gz)|1614|
|真双子叶植物|[eudicots_odb10.2020-09-10.tar.gz](https://busco-data.ezlab.org/v5/data/lineages/eudicots_odb10.2020-09-10.tar.gz)|2326|
|豆目|[fabales_odb10.2020-08-05.tar.gz](https://busco-data.ezlab.org/v5/data/lineages/fabales_odb10.2020-08-05.tar.gz)|5366|

# busco使用
## 直接在命令中设定参数【需设置的参数较少时】
`nohup busco -i genome.fa -c 10 -o busco -m geno -l busco_downloads/eudicots_odb10 --offline &`
- -i：指定需要分析的数据，组装的genome或者注释的蛋白序列或者组装的转录组dna序列;
- -m：geno/prot/tran模式；
- -c：指定线程；
- -o：指定输出文件目录名；
- -l：指定数据库
- 使用--offline离线模式

非常不建议用busco的--auto-lineage模式，这个模式在运行busco时下载数据库，网络不好时连接的失败率非常高。推荐用`-l busco_downloads/eudicots_odb10 --offline`指定下载好的本地数据库，并使用离线模式。

## 在设置文件中给出参数【需设置的参数较多时】
`nohup busco --config config.ini`
通过conda安装的这config.ini配置文件在/path/to/miniconda3/envs/busco5/config/目录下；直接复制一份到工作目录，修改使用即可。

把config.ini的示例文件中行首的分号;去掉，并把等号后的内容修改成设置的内容。

```config.ini
# This is the BUSCOv5 configuration file template.
# It is not necessary to use this, as BUSCO will use the dependencies available on your PATH by default.
# The busco run parameters can all be set on the command line. See the help prompt (busco -h) for details.
#
# To use this file for an alternative configuration, or to specify particular versions of dependencies:
# 1) edit the path and command values to match your desired dependency versions.
#    WARNING: passing a parameter through the command line overrides the value specified in this file.
#
# 2) Enable a parameter by removing ";"
#
# 3) Make this config file available to BUSCO either by setting an environment variable
#
#                   export BUSCO_CONFIG_FILE="/path/to/myconfig.ini"
#
#    or by passing it as a command line argument
#
#                   busco <args> --config /path/to/config.ini
#
[busco_run]
# Input file
;in = /path/to/input_file.fna
# Run name, used in output files and folder
;out = BUSCO_run
# Where to store the output directory
;out_path = /path/to/output_folder
# Path to the BUSCO dataset
;lineage_dataset = bacteria
# Which mode to run (genome / proteins / transcriptome)
;mode = genome
# Run lineage auto selector
;auto-lineage = True
# Run auto selector only for non-eukaryote datasets
;auto-lineage-prok = True
# Run auto selector only for eukaryote datasets
;auto-lineage-euk = True
# How many threads to use for multithreaded steps
;cpu = 16
# Force rewrite if files already exist (True/False)
;force = False
# Restart a previous BUSCO run (True/False)
;restart = False
# Blast e-value
;evalue = 1e-3
# How many candidate regions (contigs, scaffolds) to consider for each BUSCO
;limit = 3
# Metaeuk parameters for initial run
;metaeuk_parameters='--param1=value1,--param2=value2'
# Metaeuk parameters for rerun
;metaeuk_rerun_parameters=""
# Augustus parameters
;augustus_parameters='--param1=value1,--param2=value2'
# Quiet mode (True/False)
;quiet = False
# Local destination path for downloaded lineage datasets
;download_path = ./busco_downloads/
# Run offline
;offline=True
# Ortho DB Datasets version
;datasets_version = odb10
# URL to BUSCO datasets
;download_base_url = https://busco-data.ezlab.org/v4/data/
# Download most recent BUSCO data and files
;update-data = True
# Use Augustus gene predictor instead of metaeuk
;use_augustus = True

[tblastn]
path = /ncbi-blast-2.10.1+/bin/
command = tblastn

[makeblastdb]
path = /ncbi-blast-2.10.1+/bin/
command = makeblastdb

[metaeuk]
path = /metaeuk/build/bin/
command = metaeuk

[augustus]
path = /augustus/bin/
command = augustus

[etraining]
path = /augustus/bin/
command = etraining

[gff2gbSmallDNA.pl]
path = /augustus/scripts/
command = gff2gbSmallDNA.pl

[new_species.pl]
path = /augustus/scripts/
command = new_species.pl

[optimize_augustus.pl]
path = /augustus/scripts/
command = optimize_augustus.pl

[hmmsearch]
path = /usr/local/bin/
command = hmmsearch

[sepp]
path = /home/biodocker/sepp/
command = run_sepp.py

[prodigal]
path = /usr/local/bin/
command = prodigal
```

# busco结果
结果在short_summary.txt后缀文件中。

一个例子

        --------------------------------------------------
        |Results from dataset eudicots_odb10              |
        --------------------------------------------------
        |C:92.9%[S:72.4%,D:20.5%],F:1.8%,M:5.3%,n:2326    |
        |2162   Complete BUSCOs (C)                       |
        |1685   Complete and single-copy BUSCOs (S)       |
        |477    Complete and duplicated BUSCOs (D)        |
        |41     Fragmented BUSCOs (F)                     |
        |123    Missing BUSCOs (M)                        |
        |2326   Total BUSCO groups searched               |
        --------------------------------------------------

结果的解释：
使用的eudicots_odb10真双子叶植物数据库中共有2326个BUSCO groups，其中2162（92.9%）个BUSCO groups被完整比对上（包括1685个单拷贝和477个多拷贝），41个部分比对上，123个没有比对上。

通常用完整比对上的占总共的BUSCO groups的比例作为BUSCO的重要结果，越高越好，这里是92.9%=2162/2326。

# busco结果画图
在执行完毕之后，可以使用generate_plot.py画条形图，可以多个物种间比较。

首先把所有的经过BUSCO检测的物种结果short_summary.txt后缀文件放到一个文件夹（result）下；然后运行`python busco/scripts/generate_plot.py –wd result`

# 调用augustus【optional】
AUGUSTUS运行的时候需要额外设定2个环境变量，AUGUSTUS_CONFIG_PATH和BUSCO_CONFIG_FILE, 通过conda安装的这两个配置文件都在/path/to/miniconda3/envs/busco5/config目录下。

所以需要在.bashrc或.zshrc中加入下面这一行
```
export AUGUSTUS_CONFIG_PATH="/path/to/anaconda3/envs/busco5/config
export BUSCO_CONFIG_FILE="/path/to/anaconda3/envs/busco5/config/config.ini
```
实现AUGUSTUS和BUSCO的设置。

# notes
2021年06月23日开始写，07月24日今天终于完结。

