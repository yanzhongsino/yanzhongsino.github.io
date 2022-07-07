---
title: 基因组质量评估：（）统计BAM文件深度和覆盖度的软件bamdst
date: 2022-07-06
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- coverage
- depth
- mapping
- BAM
- biosoft
- bamdst

description: 介绍统计BAM文件深度和覆盖度的软件bamdst的使用和结果解释。
---

<div align="middle"><music URL></div>

# 1. bamdst简介
bamdst（https://github.com/shiquan/bamdst）是一个统计bam文件目标区域深度和覆盖度的工具，统计数据包括coverage，depth，mapping ratio。

# 2. 下载安装

```
git clone https://github.com/shiquan/bamdst.git
cd bamdst
make
./bamdst -h
```

# 3. 分析
## 3.1. 输入文件
1. sample.bed
- sample.bed文件包含分析的位置，共有三列，tab分隔。包括染色体名称，起始位置，终止位置。
- 如果计算基因组全部数据，则包含基因组所有染色体的位置。建议用genome.fa.fai索引文件转化生成`cat genome.fa.fai |awk '{print $1"\t1\t"$2}' > sample.bed`。

2. sample.bam
- sample.bam文件需要已排序（samtools sort可实现）

## 3.2. 运行
`bamdst -p sample.bed sample.bam -o ./`

- -p sample.bed：指定bed文件
- -o ./ ：输出目录

# 4. 结果
结果文件包括：

1. coverage.report

总结的统计信息，包括reads的mapping ratio，genome coverage，average depth等数据。示例：
```
## The file was created by bamdst
## Version : 1.0.9
## Files : ./sample.bam
                               [Total] Raw Reads (All reads)	86430326
                                       [Total] QC Fail reads	0
                                        [Total] Raw Data(Mb)	10773.87
                                        [Total] Paired Reads	86430326
                                        [Total] Mapped Reads	84639699
                            [Total] Fraction of Mapped Reads	97.93% # mapping ratio
                                     [Total] Mapped Data(Mb)	10550.19
                         [Total] Fraction of Mapped Data(Mb)	97.92%
                                     [Total] Properly paired	82169742
                         [Total] Fraction of Properly paired	95.07%
                                [Total] Read and mate paired	84430784
                    [Total] Fraction of Read and mate paired	97.69%
                                          [Total] Singletons	208915
                       [Total] Read and mate map to diff chr	1568692
                                               [Total] Read1	43215163
                                               [Total] Read2	43215163
                                        [Total] Read1(rmdup)	42319713
                                        [Total] Read2(rmdup)	42319986
                                [Total] forward strand reads	42323338
                               [Total] backward strand reads	42316361
                                 [Total] PCR duplicate reads	0
                     [Total] Fraction of PCR duplicate reads	0.00%
                            [Total] Map quality cutoff value	20
                       [Total] MapQuality above cutoff reads	76685486
                 [Total] Fraction of MapQ reads in all reads	88.73%
              [Total] Fraction of MapQ reads in mapped reads	90.60%
                                       [Target] Target Reads	84639699
              [Target] Fraction of Target Reads in all reads	97.93%
           [Target] Fraction of Target Reads in mapped reads	100.00%
                                    [Target] Target Data(Mb)	10457.94
                              [Target] Target Data Rmdup(Mb)	9505.01
                [Target] Fraction of Target Data in all data	97.07%
             [Target] Fraction of Target Data in mapped data	99.13%
                                      [Target] Len of region	256218203
                                      [Target] Average depth	40.82 # average depth
                               [Target] Average depth(rmdup)	37.10
                                     [Target] Coverage (>0x)	99.58% # genome coverage
                                    [Target] Coverage (>=4x)	99.12%
                                   [Target] Coverage (>=10x)	97.41%
                                   [Target] Coverage (>=30x)	68.46%
                                  [Target] Coverage (>=100x)	0.83%
                                [Target] Target Region Count	266
                                [Target] Region covered > 0x	261
                       [Target] Fraction Region covered > 0x	98.12%
                      [Target] Fraction Region covered >= 4x	97.74%
                     [Target] Fraction Region covered >= 10x	94.36%
                     [Target] Fraction Region covered >= 30x	53.01%
                    [Target] Fraction Region covered >= 100x	12.78%
                                          [flank] flank size	200
           [flank] Len of region (not include target region)	256271403
                                       [flank] Average depth	40.79
                                         [flank] flank Reads	84639699
                [flank] Fraction of flank Reads in all reads	97.93%
             [flank] Fraction of flank Reads in mapped reads	100.00%
                                      [flank] flank Data(Mb)	10452.76
                  [flank] Fraction of flank Data in all data	97.02%
               [flank] Fraction of flank Data in mapped data	99.08%
                                      [flank] Coverage (>0x)	99.54%
                                     [flank] Coverage (>=4x)	99.08%
                                    [flank] Coverage (>=10x)	97.36%
                                    [flank] Coverage (>=30x)	68.41%
                                   [flank] Coverage (>=100x)	0.83%
```

每个值具体的解释：

```
 [Total] Raw Reads (All reads) // All reads in the bam file(s).
 [Total] QC Fail reads // Reads number failed QC, this flag is marked by other software,like bwa. See flag in the bam structure.
 [Total] Raw Data(Mb) // Total reads data in the bam file(s).
[Total] Paired Reads // Paired reads numbers.
[Total] Mapped Reads // Mapped reads numbers.
[Total] Fraction of Mapped Reads // Ratio of mapped reads against raw reads.
[Total] Mapped Data(Mb) // Mapped data in the bam file(s).
[Total] Fraction of Mapped Data(Mb) // Ratio of mapped data against raw data.
[Total] Properly paired // Paired reads with properly insert size. See bam format protocol for details.
[Total] Fraction of Properly paired // Ratio of properly paired reads against mapped reads
[Total] Read and mate paired // Read (read1) and mate read (read2) paired.
[Total] Fraction of Read and mate paired // Ratio of read and mate paired against mapped reads
[Total] Singletons // Read mapped but mate read unmapped, and vice versa.
[Total] Read and mate map to diff chr // Read and mate read mapped to different chromosome, usually because mapping error and structure variants.
[Total] Read1 // First reads in mate paired sequencing
[Total] Read2 // Mate reads
[Total] Read1(rmdup) // First reads after remove duplications.
[Total] Read2(rmdup) // Mate reads after remove duplications.
[Total] forward strand reads // Number of forward strand reads.
[Total] backward strand reads // Number of backward strand reads.
[Total] PCR duplicate reads // PCR duplications.
[Total] Fraction of PCR duplicate reads // Ratio of PCR duplications.
[Total] Map quality cutoff value // Cutoff map quality score, this value can be set by -q. default is 20, because some variants caller like GATK only consider high quality reads.
[Total] MapQuality above cutoff reads // Number of reads with higher or equal quality score than cutoff value.
[Total] Fraction of MapQ reads in all reads // Ratio of reads with higher or equal Q score against raw reads.
[Total] Fraction of MapQ reads in mapped reads // Ratio of reads with higher or equal Q score against mapped reads.
[Target] Target Reads // Number of reads covered target region (specified by bed file).
[Target] Fraction of Target Reads in all reads // Ratio of target reads against raw reads.
[Target] Fraction of Target Reads in mapped reads // Ratio of target reads against mapped reads.
[Target] Target Data(Mb) // Total bases covered target region. If a read covered target region partly, only the covered bases will be counted.
[Target] Target Data Rmdup(Mb) // Total bases covered target region after remove PCR duplications. 
[Target] Fraction of Target Data in all data // Ratio of target bases against raw bases.
[Target] Fraction of Target Data in mapped data // Ratio of target bases against mapped bases.
[Target] Len of region // The length of target regions.
[Target] Average depth // Average depth of target regions. Calculated by "target bases / length of regions".
[Target] Average depth(rmdup) // Average depth of target regions after remove PCR duplications.
[Target] Coverage (>0x) // Ratio of bases with depth greater than 0x in target regions, which also means the ratio of covered regions in target regions.
[Target] Coverage (>=4x) // Ratio of bases with depth greater than or equal to 4x in target regions.
[Target] Coverage (>=10x) // Ratio of bases with depth greater than or equal to 10x in target regions.
[Target] Coverage (>=30x) // Ratio of bases with depth greater than or equal to 30x in target regions.
[Target] Coverage (>=100x) // Ratio of bases with depth greater than or equal to 100x in target regions.
[Target] Coverage (>=Nx) // This is addtional line for user self-defined cutoff value, see --cutoffdepth
[Target] Target Region Count // Number of target regions. In normal practise,it is the total number of exomes.
[Target] Region covered > 0x // The number of these regions with average depth greater than 0x.
[Target] Fraction Region covered > 0x // Ratio of these regions with average depth greater than 0x.
[Target] Fraction Region covered >= 4x // Ratio of these regions with average depth greater than or equal to 4x.
[Target] Fraction Region covered >= 10x // Ratio of these regions with average depth greater than or equal to 10x.
[Target] Fraction Region covered >= 30x // Ratio of these regions with average depth greater than or equal to 30x.
[Target] Fraction Region covered >= 100x // Ratio of these regions with average depth greater than or equal to 100x.
[flank] flank size // The flank size will be count. 200 bp in default. Oligos could also capture the nearby regions of target regions.
[flank] Len of region (not include target region) // The length of flank regions (target regions will not be count).
[flank] Average depth // Average depth of flank regions.
[flank] flank Reads // The total number of reads covered the flank regions. Note: some reads covered the edge of target regions, will be count in flank regions also. 
[flank] Fraction of flank Reads in all reads // Ratio of reads covered in flank regions against raw reads.
[flank] Fraction of flank Reads in mapped reads // Ration of reads covered in flank regions against mapped reads.
[flank] flank Data(Mb) // Total bases in the flank regions.
[flank] Fraction of flank Data in all data // Ratio of total bases in the flank regions against raw data.
[flank] Fraction of flank Data in mapped data // Ratio of total bases in the flank regions against mapped data.
[flank] Coverage (>0x) // Ratio of flank bases with depth greater than 0x.
[flank] Coverage (>=4x) // Ratio of flank bases with depth greater than or equal to 4x.
[flank] Coverage (>=10x) // Ratio of flank bases with depth greater than or equal to 10x.
[flank] Coverage (>=30x) // Ratio of flank bases with depth greater than or equal to 30x.
```

2. chromosomes.report

每条染色体的depth和coverage信息，示例：
```
#Chromosome	   DATA(%)	 Avg depth	    Median	 Coverage%	  Cov 4x %	 Cov 10x %	 Cov 30x %	Cov 100x %
  MCscaf061	    0.00	   42.69	     41.0	  100.00	  100.00	   99.30	   86.09	    0.00
  MCscaf062	    0.02	   66.31	     37.0	  100.0099.85	   99.26	   71.85	    0.40
```

3. insertsize.plot

推断的insert size分布。示例：
```
0	0	0.000000	41302164	1.000000
1	0	0.000000	41302164	1.000000
2	169	0.000004	41301995	0.999996
3	154	0.000004	41301841	0.999992
4	179	0.000004	41301662	0.999988
5	220	0.000005	41301442	0.999982
```

4. uncover.bed

文件包含sample.bam在sample.bed上bad covered或uncovered region的区域。示例：
```
MCscaf062	3308	3324
MCscaf062	13172	13219
MCscaf063	1966	1988
MCscaf064	154	177
MCscaf065	4380	4388
MCscaf065	15265	15278
MCscaf065	49992	50000
```

5. depth_distribution.plot

示例：
```
0	1080969	0.004219	255137234	0.995781
1	375560	0.001466	254761674	0.994315
2	376656	0.001470	254385018	0.992845
3	420120	0.001640	253964898	0.991206
4	471142	0.001839	253493756	0.989367
5	548760	0.002142	252944996	0.987225
```

6. depth.tsv.gz

文件包含输入的sample.bed的每个位置的三种深度：
- raw depth：没有过滤的从bam文件直接提取的深度。coverage.report文件是用raw depth统计得到的。
- rmdup depth：过滤掉duplicated reads，secondary alignment reads，low map quality reads(mapQ<20)后计算的depth。开发者说类似`samtools depth`的结果，但应该是raw depth类似`samtools depth`，rmdup depth类似`samtools mpileup`才更准确。如果想用rmdup depth统计coverage.report，运行时加上参数“--use_rmdup”。
- coverage depth：考虑deletion区域的raw depth，所以值会大于或等于raw depth的值。

7. region.tsv.gz

文件包含输入的sample.bed的每个区域的average depth，median depth，coverage。


# 5. references
1. https://github.com/shiquan/bamdst