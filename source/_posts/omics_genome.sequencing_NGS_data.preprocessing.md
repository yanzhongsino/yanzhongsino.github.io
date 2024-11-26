---
title: 二代测序数据的前处理
date: 2024-11-26
categories:
- omics
- genome
- genome sequencing
- next generation sequencing
tags:
- quality control
- QC
- genome sequencing
- next generation sequencing
- NGS
- NCBI
- SRA
- SRA Toolkit
- prefetch
- fastq-dump
- fasterq-dump
- fastp
- trimmomatic
- fastqc
- fastuniq
- fastq
- genome survey
- data preprocessing

description: 对二代测序进行预处理。包括从NCBI下载SRA格式数据，把SRA格式转为fastq格式。对二代测序的下机数据的质控，包括fastp去除接头和低质量序列，fastqc获取质量报告，trimmomatic去除接头和低质量序列，fastuniq为genome survey等情况去除重复序列。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=284982&auto=1&height=32"></iframe></div>

# 1. 二代测序数据（又叫高通量测序数据）
1. 二代测序数据（包括全基因组测序数据，转录组测序数据，常用Illumina PE reads）的下机数据大部分是raw reads。
2. 如果是送公司测序，收到的很大可能是fastq格式的raw reads。如果是从NCBI等公共数据库下载，则可能是SRA格式，需要转换格式。
3. 通常在下游分析前都需要先进行质控，包括：
- 去掉接头（adapter）序列；
- 去掉质量较低的reads
- 去掉被细菌或其他生物污染产生的reads
1. 进行质控过滤低质量reads后的序列通常被称为clean reads。

# 2. NCBI下载二代数据raw reads和预处理
如果是使用NCBI等公共数据库的raw reads数据，则需要先下载SRA格式文件和转换为fastq格式文件。
## 2.1. 快速上手
找到NCBI数据的SRR序列号，然后prefetch下载SRA格式数据，fasterq-dump转SRA格式为fastq格式。
- `prefetch SRR1234567 -p`
- `fasterq-dump --split-3 ./SRR1234567 -p -e 12`

## 2.2. SRA和SRA Toolkit介绍
1. SRA格式
- SRA（Sequence Read Archive）格式是NCBI专用的储存高通量测序数据的格式，可以处理各种类型的测序技术生成的数据，包括Illumina、Ion Torrent、454、PacBio等。
2. SRA Normalized和SRA Lite格式的区别
- SRA Normalized是SRA的标准化格式，大部分情况下使用这个格式。
- SRA Lite是把SRA Normalized的质量分数（quality scores）进行简化，将碱基质量得分分为了pass和reject两种，pass统一给分为30，而reject统一给分为3。 
- 下载得到的SRA Normalized文件名为SRR1234567；下载得到的SRA Lite文件名为SRR1234567.lite.1（文件更小）。
- SRA转为fastq格式的操作对两种格式都有效。
3. SRA Toolkit
- SRA Toolkit是由NCBI提供的一组工具，用于访问、下载和处理存储在SRA（Sequence Read Archive）中的高通量测序数据。这个工具套件允许用户从SRA数据库中检索数据、将数据格式转换为更常用的格式（如FASTQ）以及其他数据处理任务。
- SRA Toolkit软件的使用manual： https://github.com/ncbi/sra-tools/wiki
4. SRA Toolkit下载和安装
- SRA Toolkit下载：在 https://github.com/ncbi/sra-tools/wiki/01.-Downloading-SRA-Toolkit 找到对应系统的软件包下载。
- SRA Toolkit解压缩即可使用：`tar -xvzf sratoolkit.current-ubuntu64.tar.gz`
5. SRA Toolkit的命令
- SRA Toolkit包括下载SRA数据的命令`prefetch`,将SRA格式的数据转换为FASTQ格式的命令`fastq-dump`,`fasterq-dump`，检索SRA数据文件中的元数据的`vdb-dump`,将SRA文件直接转换为SAM格式（如果记录在数据库中有对映/SAM信息）的`sam-dump`。
## 2.3. 下载SRA文件和转换格式
### 2.3.1. 查找SRR编号
1. 在NCBI网站 https://www.ncbi.nlm.nih.gov/sra 搜索物种名等信息，获取SRR编号；
2. 选择多个搜索结果，然后Send to-File-RunInfo下载SraRunInfo.csv表格文件，里面有非常完整的上传人填写的测序相关信息，包括SRA Lite格式的download_path；
3. 选择多个搜索结果，然后Send to-File-Acdession List下载SraAccList.csv表格文件。
4. 点击进入一个搜索结果，然后点击SRR号，再点击Data access栏，在SRA archive data栏就能看到两种格式的SRA数据的下载网址。

### 2.3.2. 下载SRA文件
1. prefetch调取下载
- 先在NCBI查询需要下载的数据的SRR编号，再使用编号信息下载SRA文件:`prefetch SRR1234567 SRR7654321`命令会下载SRA Normalized格式文件，支持多个SRR编号参数，依次下载。
- 多个SRA数据下载，也可以把SRR编号保存在文件中，一个编号一行（同SraAccList.csv表格文件）。然后用--option-file参数指定保存了编号的文件，来依次下载多个SRA数据：`prefetch --option-file SraAccList.csv`
- 其他参数：
  - -X|--max-size: 设置最大下载文件大小，默认20G。
  - -p|--progress: 显示下载进度。
  - --eliminate-quals：下载SRA-Lite格式文件。

2. wget下载
- 复制下载网址之后用wget下载：`wget -c http:url`

### 2.3.3. SRA格式转为fastq格式
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
-`fasterq-dump --split-3 ./SRR1234567 -p -e 12`
- 参数：
  - -s|--split-spot: 将双端测序分为两份,放在一个文件中。
  - -S|--split-files: 将双端测序分为两份,放在两个文件，对于一方有而一方没有的reads直接丢弃。
  - -3|--split-3 : 将双端测序分为两份,放在两个文件，对于一方有而一方没有的reads（unpaired reads）会单独放在一个文件夹里。
  - -p：显示进程。
  - -e 24：指定线程，默认6。

# 3. 质控 （rawreads2cleanreads）
不管从哪种途径获得了fastq格式的rawdata后，就需要进行质控操作。
常用的二代测序数据质控软件包括fastp,Trimmonmatic,FastQC,Cutadapt,BBMap,BBduk。
这里介绍fastp，Trimmonmatic,FastQC三个软件，都支持fq.gz压缩文件。

## 3.1. fastp快速方案（fastp速度=trimmomatic速度*5）
fastp可以提供快速高效的FASTQ文件预处理和质控，包括过滤低质量读段、去除接头、质量剪切等功能，同时可以提供质量报告。
1. 命令

`time fastp -c -l 50 -q 20 -w 8 -i sample_1.raw.fq.gz -I sample_2.raw.fq.gz -o sample_1.clean.fq.gz -O sample_2.clean.fq.gz -h sample_fastp.html -j sample_fastp.json`

2. 常用参数-输入输出
- -i, -I 分别指定双端测序的raw reads文件，支持.fq.gz格式。
- -o, -O 分别指定质控后生成的双端测序的clean reads文件，支持.fq.gz格式。
- -h 或 --html: 生成质量控制报告的HTML文件。
- -j 或 --json: 生成质量控制报告的JSON文件。
3. 常用参数-质控
- -c, --correction：在overlapped区域执行碱基校正（base correction），只对PE数据有效, 默认不执行。
- -l 50：设置保留reads的最小长度，默认15。
- -q 20：设置碱基质量的最小值，默认phred quality >=Q15。
- -w 8：设置线程，默认2。
4. 其他参数
- -u 或 --unqualified_percent_limit: 设置允许的低质量碱基的最大百分比，超过该比率的读取将被丢弃。
- -n 或 --n_base_limit: 设置允许的N碱基的最大数目，超过该数目的读取将被丢弃。
- -L 或 --length_limit: 设置保留读取的最大长度，超过该长度的读取将被丢弃。
- -f 或 --trim_front: 去除序列开头的若干个碱基。
- -t 或 --trim_tail: 去除序列末尾的若干个碱基。
- -b 或 --cut_by_quality3: 启用右端质量控制剪切，去除序列末尾低质量碱基。
- -a 或 --adapter_sequence: 指定单端测序的适配子序列。
- -A 或 --adapter_sequence_r2: 指定双端测序第二条链的适配子序列。
- --detect_adapter_for_pe: 自动检测并剪切双端测序的适配子。
- -W 或 --overlap_len_require: 对于双端数据，设置最小重叠长度以进行合并。

## 3.2. Trimmomatic + FastQC 经典方案
Trimmomatic则用于reads修剪和过滤，包括接头去除和质量剪切，但无法提供质控报告。FastQC可以提供详细的质量报告，包括序列质量评分、GC含量分布、序列长度分布等。许多文章都使用这个两个软件组合进行质控和报告的获取。
### 3.2.1. FastQC 获取raw reads的质量报告
1. FastQC 运行
- `time fastqc sample_1.raw.fq.gz -t 8 -o sample`
- `time fastqc sample_2.raw.fq.gz -t 8 -o sample`
2. 常用参数
- -o：指定输出目录，在目录下生成<input_file_name>_fastqc.html和<input_file_name>_fastqc.zip两个质量报告文件。
- -f：指定输入文件格式，如fastq，bam。默认通过文件名检测。
- -t：指定线程。
- -c：提供污染物序列的文件。FastQC可根据此文件进行污染物检测。
- -a: 提供接头序列的文件，FastQC使用这些信息检测接头污染。
4. 质量报告：FastQC为每个输入文件生成一个压缩的<input_file_name>_fastqc.zip文件和一个可浏览的<input_file_name>_fastqc.html报告，报告中包含一系列质量指标，包括：
- Per base sequence quality: 每个碱基位置的质量评分。
- Per sequence GC content: 存在于读段中的GC含量的分布。
- Per base N content: 每个位置N碱基的比例。
- Sequence Length Distribution: 序列长度的分布。
### 3.2.2. Trimmomatic 质控
Trimmomatic 可以去除接头，低质量序列和被污染序列。
1. trimmomatic 命令
`time trimmomatic PE -threads 8 -phred33 sample_raw.R1.fq.gz sample_raw.R2.fq.gz sample_clean.R1.fq.gz sample.unpaired.R1.fq.gz sample.clean.R2.fq.gz sample.unpaired.R2.fq.gz ILLUMINACLIP:adapters.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36`
trimmomatic是一个对Illumina NGS data进行trim的工具
2. 常用参数
- PE/SE 设定对Paired-End或Single-End的reads进行处理。
- -threads 设置多线程运行数
- -phred33/phred64 选择碱基的质量格式，默认是phred64，目前大部分质量格式都为phred33。
3. 输入输出文件
- PE模式，后面提供两个输入文件（sample_raw.R1.fq.gz sample_raw.R2.fq.gz），对应四个输出文件（sample_clean.R1.fq.gz sample.trim.R1.fq.gz sample.clean.R2.fq.gz sample.trim.R2.fq.gz）
- 输出文件中sample_clean前缀的文件是质控之后的clean reads文件，sample.unpaired前缀的文件是质控过程中未配对的reads文件。
- SE模式，后面提供一个输入文件，对应两个输出文件。
4. 质控参数
- ILLUMINACLIP:adapters.fa:<seed mismatches>:<palindrome clip threshold>:<simple clip threshold>： 用于去除接头（adapters）。
  - 提供一个包含接头序列的 FASTA 文件（adapters.fa），用于识别接头序列。
  - <seed mismatches>比对adapter序列时，允许的最大mismatch错配数；
  - <palindrome clip threshold>识别回文接头的匹配碱基数阈值。设置为30代表PE的一对read同时和adapter序列比对，匹配度超30%，就认为这对read含有adapter，在对应位置切除）；
  - <simple clip threshold>识别简单接头的匹配碱基数阈值。设置为10代表单条read与adapter比对的匹配度超10%，就认为这条read含adapter，在对应位置切除。
- LEADING:<quality>： 去除首端低于指定质量值的碱基，直至遇到一个质量合格的碱基。
- TRAILING:<quality>： 去除末端低于指定质量值的碱基，直至遇到一个质量合格的碱基。
- SLIDINGWINDOW:<windowSize>:<requiredQuality>： 用滑动窗口法检查reads质量，并设置滑窗大小和质量要求。窗口内的平均质量低于指定值时，包括滑窗内及往后的所有碱基都被切除。
- MINLEN:<length>：设置保留的最短reads长度，低于此长度的序列将被丢弃。
- CROP:<length> 保留reads到指定的长度。
- HEADCROP:<length> 在reads的首端切除指定的长度。
- AVGQUAL:20 最低质量阈值。被切除后的read平均质量低于20，则丢弃此条read。
### 3.2.3. FastQC 获取clean reads的质量报告
FastQC 获取clean reads的质量报告，可以与raw reads的报告对比着看。
- `time fastqc sample_clean.R1.fq.gz -t 8 -o sample_clean`
- `time fastqc sample_clean.R2.fq.gz -t 8 -o sample_clean`

# 4. 去除重复序列（cleanreads2uniqreads）
## 4.1. 去重操作的背景
1. 测序过程中（二代测序主要是PCR环节引入）可能带来重复片段Duplication，通常重复片段占比<15%。
[Duplication占比问题的解释](http://blog.sciencenet.cn/blog-3406804-1215719.html)
2. 去重一般操作比对到基因组上并排序完成的bam文件，利用基因组的位置信息进行去重，效率较高。
3. 若没有参考基因组的情况，比如**genome survey分析前的数据预处理，可以直接对clean reads进行去重**。
## 4.2. Fastuniq去重操作
1. 命令

```shell
realpath sample_1.clean.fq sample_2.clean.fq >input.txt
time fastuniq -i input.txt -o sample_1.fastuniq.clean.fq -p sample_2.fastuniq.clean.fq -t q
```

2. 参数
- -i：输入文件列表，包含多个 FASTQ 文件的列表文件，一行一个文件名。
- -o：输出去重后的第一个文件。
- -p：输出去重后的第二个文件。
- -t：指定输入文件的格式类型（默认q），q:两个fastq文件; f:两个fasta文件; p:一个fasta文件。
3. notes
- fastuniq不支持直接读取压缩格式*.fq.gz作为输入文件，需要先解压；输出结果也不支持压缩。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>