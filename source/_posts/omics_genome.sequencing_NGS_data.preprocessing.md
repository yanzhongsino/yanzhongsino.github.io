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
4. 进行质控过滤低质量reads后的序列通常被称为clean reads。

# 2. NCBI下载二代数据raw reads和预处理
如果是使用NCBI等公共数据库的raw reads数据，则需要先下载SRA格式文件和转换为fastq格式文件，使用NCBI官方工具SRA Toolkit里的prefetch和fasterq-dump命令来下载和转换。
## 2.1. 快速上手
找到NCBI数据的SRR序列号，然后prefetch下载SRA格式数据，fasterq-dump转SRA格式为fastq格式。
- `prefetch SRR123456 SRR654321 -p`
- `fasterq-dump --split-3 ./SRR1234567 -p -e 12`

# 3. 质控 （rawreads2cleanreads）
不管从哪种途径获得了fastq格式的rawdata后，就需要进行质控操作。
常用的二代测序数据质控软件包括fastp,Trimmonmatic,FastQC,Cutadapt,BBMap,BBduk。
这里介绍fastp，Trimmonmatic,FastQC三个软件，都支持fq.gz压缩文件。

## 3.1. fastp快速方案（fastp速度=trimmomatic速度*5）
fastp可以提供快速高效的FASTQ文件预处理和质控，包括过滤低质量读段、去除接头、质量剪切等功能，同时可以提供质量报告。
1. 命令
- `time fastp -c -l 50 -q 20 -w 8 --detect_adapter_for_pe -i sample_1.raw.fq.gz -I sample_2.raw.fq.gz -o sample_1.clean.fq.gz -O sample_2.clean.fq.gz -h sample_fastp.html -j sample_fastp.json`
2. 常用参数-输入输出
- -i, -I 分别指定双端测序的raw reads文件，支持.fq.gz格式。
- -o, -O 分别指定质控后生成的双端测序的clean reads文件，支持.fq.gz格式。
- -h 或 --html: 生成质量控制报告的HTML文件。
- -j 或 --json: 生成质量控制报告的JSON文件。
- -h和-j的输出文件也可以用来检验运行有没有被中断。如果运行中断，会生成clean.fq.gz，但不会生成html和json文件。
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
- -a 或 --adapter_sequence, --adapter_sequence_r2: 指定单端/双端测序的接头序列。
- --detect_adapter_for_pe: 自动检测接头序列仅对单端reads是默认执行的，用这个参数自动检测双端测序的接头序列。
- -A: 默认执行adapter trimming（去除接头序列），用-A参数则不执行。
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