---
title: genome
date: 2020-07-10 09:50:00
categories: bio，genome

---  

# 三代测序数据基因组组装注释流程
三代测序平台主要有Pacific Biosciences的单分子实时测序和Oxford Naropore公司的单纳米孔测序两种平台。  


## 基因组survey
在基因组大规模测序或者正式组装之前，首先构建DNA小片段文库进行中低深度的二代测序，使用PE文库测序所得的reads信息进行基因组Survey分析以初步评估基因组特征,包括基因组大小genome size，杂合度heterozygosity，重复序列比例，GC含量等。
* 基因组大小：基因组越大，测序花的钱越多
* 简单基因组: 杂合度低于0.5%, GC含量在35%~65%, 重复序列低于50%
* 二倍体普通基因组: 杂合度在0.5%~1.2%中间，重复序列低于50%；或杂合度低于0.5%，重复序列低于65%
* 高复杂基因组: 杂合度>1.2% 或 重复率大于65%

基因组survey分析核心是**k-mer分析。**

### k-mer分析前数据处理
#### 质控和过滤
对下机的低深度二代raw reads进行质控和过滤，去除质量低的序列和接头adapters序列，获得clean reads
```
$fastp -c -l 50 -w 8 -i sample_1.raw.fq.gz -I sample_2.raw.fq.gz -o sample_1.clean.fq.gz -O sample_2.clean.fq.gz -h sample_fastp.html -j sample_fastp.json
# 生成*_1.clean.fq.gz、*_2.clean.fq.gz结果文件，和*.html可视化QC报告文件, *.json JSON版报告文件
```
#### 去重
去除测序过程中（二代测序主要是PCR环节引入）带来的重复片段Duplication，通常重复片段占比<15%。
[Duplication占比问题的解释](http://blog.sciencenet.cn/blog-3406804-1215719.html)

*p.s. 去重一般操作比对到基因组上并排序完成的bam文件，利用基因组的位置信息进行去重，效率较高。若没有参考基因组的情况，比如这里的genome survey分析前去重，可以直接比对fq文件实现。*
```
$cat input_list.txt
path_to_sample_1.clean.fq
path_to_sample_2.clean.fq
$fastuniq -i input_list.txt -o sample_1.fastuniq.clean.fq -p sample_2.fastuniq.clean.fq
# 去重后生成sample_*.fastuniq.clean.fq两个文件
# fastuniq不支持压缩格式*.fq.gz输入文件
```
### k-mer分析
分为k-mer频数统计和基因组特征评估两步，软件KmerGenie一行命令同时实现两步，软件gce两行命令分别实现两步，jellyfish+genomescope两个软件分别实现两步。KmerGenie，gce和jellyfish软件第一步获取的频数分布表，都可用于genomescope和gce软件第二步骤的分析。

由于gce第一步骤支持的最大k-mer频数为255，大于255的数据被合并；而jellyfish统计到10000行，预估结果会更为准确。Genomescope对于高重复序列的基因组统计的基因组大小会偏小，建议max kmer coverage设置成10000。

建议使用jellyfish+genomescope/gce或者KmerGenie进行k-mer分析。

[k-mer分析介绍](http://blog.sciencenet.cn/blog-3406804-1162384.html)
#### k-mer频数统计
多个软件可以实现，目的是获取k-mer频数分布表。

k-mer长度一般选择17/21，另一个参数需注意和设定，单倍体模式还是杂合模式，可以两种模式都分析，查看差别。
1. [jellyfish](http://blog.sciencenet.cn/blog-3406804-1161522.html)
```
$jellyfish count -m 17 -s 1G -t 8 -o k17.jf -C sample_1.fastuniq.clean.fq sample_2.fastuniq.clean.fq
# k-mer计数，当前工作路径下生成结果文件k17.jf，k-mer长度17，存储用的hash表大小为1G，线程8, -C对DNA正负链都进行统计，表示考虑DNA正义与反义链，遇到反义kmer时，计入正义kmer频数中。结果文件名为k17.jf。jellyfish不支持压缩格式*.fq.gz输入文件
$jellyfish histo k17.jf > k17.histo
# 统计 k-mer 计数得到 k-mer 频数分布表
```
值得注意的是，当输入PE双端测序数据时，-C参数下jellyfish只会统计一半数据量。

2. [KmerGenie](http://blog.sciencenet.cn/blog-3406804-1159967.html)
```
$cat fastq_list.txt
path_to_sample_1.fastuniq.clean.fq
path_to_sample_2.fastuniq.clean.fq
$kmergenie fastq_list.txt -o ./sample -l 17 -k 121 -s 10 -t 4 > sample.log1.txt 2> sample.log2.txt
# 默认单倍体模式，以k-mer长度17为起始，121为终止，10为间隔逐一测试；程序运行线程数4。结果输出在当前路径下，以sample为结果文件前缀名。“sample.log1.txt”和“sample.log2.txt”分别为程序运行时的正确/错误输出日志。
```
生成结果报告文件*_report.html，报告开头以折线图的形式展示出在每种长度k-mer下，估算的基因组大小，同时给出了“最佳k-mer”选择数值（其实就是将评估基因组总大小最高的那个k-mer值判定为“最佳k-mer”，为基因组组装时k-mer的选择提供参考）。

生成各k-mer取值下的频数分布表*.histo和对应的频数分布图*.histo.pdf，以及所有k-mer取值的总计*.dat和*.dat.pdf

3. [gce](http://blog.sciencenet.cn/blog-3406804-1161524.html)
```
$kmer_freq_hash -k 17 -L 150 -l fastq_list.txt -t 4 -o 0 -p sample &> sample.kmer.log
# k-mer长度为17，设定统计用的reads最大长度150bp，线程4，为节省时间不输出k-mer序列，结果文件前缀名sample
```
结果文件k-mer频数分布表“sample.freq.stat”和运行log文件“sample.kmer.log”
sample.freq.stat文件只统计到第255行，第255行之后的数据合并至第255行，表示k-mer出现频数>=255的片段总数。

sample.kmer.log为程序运行的日志文件，同时对测序数据进行了简要统计。该文件的最下方，统计了k-mer片段总数、k-mer种类数、k-mer平均频数、碱基总数、reads平均长度、基因组大小的粗略估计等信息。

频数分布表sample.histo/sample.kmer.freq.stat文件有两列，第一列是k-mer频数，第二列是频数对应的k-mer数量，预期是泊松分布，在k-mer频数中间有k-mer数量的峰值；频数小于5的那几列对应的k-mer数量高是测序错误造成的，频数最大的那列对应的k-mer数量高是把所有大于该列的频数进行合计数量造成的，两端都可以忽略。

#### 基因组特征评估
根据获取的频数分布表，进行基因组特征的评估，多个软件可以实现。

R脚本绘制k-mer频数分布曲线初步查看基因组特征
```R
$R
#R 脚本示例
kmer <- read.table('sample.histo')
kmer <- subset(kmer, V1 >=5 & V1 <=500) #对频数范围5-500的数据进行绘制 
Frequency <- kmer$V1
Number <- kmer$V2
png('kmer_plot.png')
plot(Frequency, Number, type = 'l', col = 'blue')
dev.off()
```
获得kmer_plot.png为频数分布曲线，可查看曲线峰值对基因组大小进行计算和预估。

1. genomescope

使用[genomescope网页版](http://qb.cshl.edu/genomescope/)上传第一步获得的kmer频数分布表histo文件，设置参数Kmer length为第一步选择的k-mer长度值，这里是17；参数Read length为序列读长，一般为150；最后一个参数Max kmer coverage建议修改成10000，以统计更准确。

结果显示预估的基因组大小，杂合度，重复率等信息。

2. KmerGenie

生成结果报告文件*_report.html展示了在每种长度k-mer下，估算的基因组大小，同时给出了“最佳k-mer”选择数值，给组装基因组提供参考。

3. gce

峰值对应的k-mer频数(假设为n)。
```
gce -f sample.freq.stat -c n -b 1 -H 0 -m 1 -D 8 -M 256 > sample.gce.stat 2> sample.gce.log
# -c n频数峰值为n；-b 1数据有bias；-H 0单倍体模式；-m 1估算模型使用连续型；-D 8期望值精度；-M 256支持的最大k-mer频数，若输入jellyfish获得的sample.histo，则设置-M 10001。
```
结果文件sample.gce.stat和sample.gce.log，在log文件最后记录了基因组特征评估结果，包括估算的kmer平均深度cvg，基因组大小genome_size。

## pacbio三代测序组装
### 组装前数据准备
pacbio三代测序数据下机文件为sample.subreads.bam格式，sample.subreads.bam.pbi是bam文件的index文件

把下机bam文件转换成fasta文件格式（bam文件中碱基质量数据都为！，所以转换成fastq会无质量数据，建议转化成fasta）
```
samtools bam2fq sample.subreads.bam | seqtk seq -A > sample.subreads.fa
```

### 组装assemble
**组装软件有canu, mecat2，nextdenovo。**

canu是老牌组装软件，准确率高但速度慢，nextdenovo速度比canu快，mecat2最快，服务器不强悍推荐mecat2。一般多个组装软件共同使用，并比较差异。
#### canu2.0
canu installation
```
wget https://github.com/marbl/canu/releases/download/v2.0/canu-2.0.Linux-amd64.tar.xz # 下载二进制包
tar -xJf canu-2.0.*.tar.xz # 解压缩和打开包
export PATH=./canu-2.0/*/bin:$PATH # 安装完成，添加到环境变量，目录下canu为主执行文件
```
canu run
```
canu -p sample -d sample_pacbio -s setting.txt genomeSize=400m -pacbio sample.subreads.fasta
#-p sample 结果文件的前缀prefix；-d sample_pacbio 结果文件存放目录命名；-s setting.txt 设置文件，更多的参数设置可在设置文件中进行； genomeSize预估的基因组大小；-pacbio pacbio测序获得的reads，canu会按correction,trimming,assembly三个步骤依次运行。
```
setting.txt文件格式如下:
```
maxThreads=26 # 最大线程
maxMemory=500g #最大内存
minReadLength=2000 # 只使用长度大于设置值的序列
correctedErrorRate=0.050 # 设置纠错后read之间最大期望差异碱基数，组装时需多次调整,推荐0.035-0.05
minOverlapLength=500 # 增加可降低假阳性的overlap
corOutCoverage=200 #
corMinCoverage=5 #
ovsMemory=16G
ovbConcurrency=15
ovsConcurrency=15
```
[Canu 2.0 及以下版本](https://www.jianshu.com/p/c1aeeae77cb5)组装结果contigs文件内包含了bubbles，开发者建议用purge_dups去冗余之后在进行下一步分析。

#### mecat2
原始数据纠错
mecat.pl correct setting.txt
组装纠错后的reads
mecat.pl assemble setting.txt
结果文件
4.result/4-fsa/contigs.fasta

mecat2 installation
```
mkdir path/mecat2 && cd path/mecat2 #创建目录
export MECAT_PATH=path/mecat2 && echo ${MECAT_PATH} #创建变量
git clone https://github.com/xiaochuanle/MECAT2.git #下载mecat2软件
cd MECAT2
make #编译
cd ..
export PATH=${MECAT_PATH}/MECAT2/Linux-amd64/bin:$PATH # 安装完成，添加到环境变量，目录下mecat.pl为主执行文件
```

mecat2 run
mecat2的工作流程包括纠错（correction），修剪（trimming），组装（assemble）3个stage阶段。

1.准备设置文件。参数都在设置文件中设置，设置文件setting.txt内容如下:
```
PROJECT=result #项目名称，在运行目录下生成项目名称为名的目录，存放结果文件
RAWREADS=/path/sample.subreads.fastq.gz #数据目录
GENOME_SIZE=60000000 #基因组大致的大小，单位bp，60000000代表60Mbp
THREADS=12 #线程数
MIN_READ_LENGTH=2000 #用于correct和trim阶段的reads的最低长度值，数据质量好可设置成1000/2000
CNS_OVLP_OPTIONS="" #correction阶段检测候选重叠的参数，传给mecat2pw
CNS_OPTIONS="" #原始reads的correct参数
CNS_OUTPUT_COVERAGE=30 #选择30X覆盖度的最长corrected的reads进行trim和assemble，传递给mecat2elr，60Mbp的基因组30X的reads数据量为1800Mbp，常用20-40
TRIM_OVLP_OPTIONS="" #trimming阶段检测重叠的参数，传给v2asmpm,-B输出二进制格式比对结果，若无此选项，则输出文本格式
ASM_OVLP_OPTIONS="" #assemble阶段用于检测重叠的参数,传递给v2asmpm.sh，可用默认参数
FSA_OL_FILTER_OPTIONS="--max_overhang=-1 --min_identity=-1" #过滤重叠的参数，传递给fsa_ol_filter,-1表示让程序自己决定
FSA_ASSEMBLE_OPTIONS="" #assemble阶段参数，传递给v2asmpm
USE_GRID=false #是否有多个计算节点
GRID_NODE=0 #若有多个计算节点，设置用到的计算节点数
CLEANUP=1 #运行结束后删除mecat2的中间文件，大基因组最后设置为1，小基因组可以设置为0保留中间文件
```
2. 原始数据纠错
```
mecat.pl correct setting.txt
```
3. 组装纠错后的reads
```
mecat.pl assemble setting.txt
```
4. 结果文件

result/1-consensus/cns_reads.fasta #corrected后的reads
result/1-consensus/cns_final.fasta #corrected后最长30X用于trim的reads
result/2-trim_bases/trimReads.fasta #trimmed reads
result/4-fsa/contigs.fasta #assembled的contigs

#### nextDenovo
[nextDenovo ref.](http://blog.sciencenet.cn/blog-3406804-1204832.html)

nextDenovo installation
```
# 下载程序，下载后即是已编译好的二进制版，无需安装，可直接使用
wget https://github.com/Nextomics/NextDenovo/releases/download/v2.1-beta.0/NextDenovo.tgz
tar -vxzf NextDenovo.tgz
# 添加可执行权限
cd NextDenovo
chmod -R 755 *
# 添加至环境变量
export PATH=~/software/NextDenovo/:$PATH
export PATH=~/software/NextDenovo/bin/:$PATH
# 软件运行需要python2.7，提示缺少模块时安装即可；不支持python3
```
nextDenovo run
```
nextDenovo run.cfg
```
配置文件run.cfg的格式如下:
```
[General]                # global options
	job_type = local           # 运行环境 [local, sge, pbs...]. (default: sge)
	job_prefix = nextDenovo  # 作业名称 prefix tag for jobs. (default: nextDenovo)
	task = all               # 任务类型 task need to run [all, correct or assemble]. (default: all)
	rewrite = yes             # 是否覆盖 overwrite existed directory [yes, no]. (default: no)
	deltmp = yes             # 是否删除中间文件 delete intermediate results. (default: yes)
	rerun = 3                # 重新运行未完成作业，直到完成或达到${rerun}循环 re-run unfinished jobs untill finished or reached ${rerun} loops, 0=no. (default: 3)
	parallel_jobs = 10       # 并行运行的任务数 number of tasks used to run in parallel. (default: 10)
	input_type = raw         # 输入数据类型input reads type [raw, corrected]. (default: raw)
	input_fofn = input.fofn  # 指定文本文件，文件中写入输入数据的路径，多个数据时一个路径占一行，input file, one line one file. (required)
	workdir = 01.workdir     # 工作路径 work directory. (default: ./)
	usetempdir = /tmp/test   # temporary directory in compute nodes to avoid high IO wait. (default: no)
	nodelist = avanode.list.fofn
	                         # a list of hostnames of available nodes, one node one line, used with usetempdir for non-sge job_type.
	cluster_options = auto
	                         # a template to define the resource requirements for each job, which will pass to DRMAA as the nativeSpecification field.

	[correct_option]         # options using only in corrected step.
	read_cutoff = 1k         # 过滤长度低于该设定阈值的reads filter reads with length < read_cutoff. (default: 1k)
	seed_cutoff = 25k        # 选择长度大于该设定值以上的reads进行校正 minimum seed length. (required)
	seed_cutfiles = 10       # split seed reads into ${seed_cutfiles} subfiles. (default: ${pa_correction})
	blocksize = 10g          # block size for parallel running. (default: 10g)
	pa_correction = 15       # 用于校正任务的并行运行数 number of corrected tasks used to run in parallel. (default: 15)
	minimap2_options_raw = -x ava-ont -t 10   
	                         # 用于寻找raw reads和corrected reads之间重叠，传递给子程序minimap2执行； minimap2 options, used to find overlaps between raw reads and set PacBio/Nanopore read overlap, see minimap2-nd -h for details. (required)
	sort_options = -m 5g -t 2 -k 50   
	                         # 根据与给定seed的匹配数对冗余重叠进行排序和删除，传递给子程序ovl_sort执行； sort options, see ovl_sort -h for details.  
	correction_options = -p 10            
	                         # 纠错程序的一些参数设置
                             # -p, --process, set the number of processes used for correcting. (default: 10)
	                         # -b, --blacklist, disable the filter step and increase more corrected data.
	                         # -s, --split, split the corrected seed with un-corrected regions. (default: False)
	                         # -fast, 0.5-1 times faster mode with a little lower accuracy. (default: False)
	                         # -dbuf, Disable caching 2bit files and reduce ~TOTAL_INPUT_BASES/4 bytes of memory usage. (default:False)
	                         # -max_lq_length, maximum length of a continuous low quality region in a corrected seed, larger max_lq_length will produce more corrected data with lower accuracy. (default: auto [pb/1k, ont/10k])

	[assemble_option]
	random_round = 20        # 随机组装参数的数量，会基于每一随机参数做一次组装，避免默认参数效果不好； number of random parameter sets. (default: 10)
	minimap2_options_cns = -x ava-ont -t 8 -k17 -w17 
	                         # 寻找corrected reads之间的重叠，传递给子程序minimap2执行，详见minimap2-nd -h； minimap2 options, used to find overlaps between corrected reads. (default: -k17 -w17)
	nextgraph_options = -a 1 # 组装程序的一些参数设置； nextgraph options, see here for details.
```

nextDenovo最终的组装序列：
03.ctg_graph/01.ctg_graph.sh.work/ctg_graph00/nextgraph.assembly.contig.fasta

### 基因组去冗余
[purge_dups](http://xuzhougeng.top/archives/remove-redundancy-using-purge-dups)

canu2.0组装结果



### 基因组校正polish
pacbio三代数据组装好的基因组polish（校正）
由于三代测序错误率较高，需要用原始下机的rawdata（二代或者三代）对组装好的基因组进行碱基校正（组装软件通常在组装前也有校正环节，但组装后仍需校正，以减少不同软件组装结果的差异），组装后的校正环节称为polish，用于polish的软件有NextPolish（可用于二代或三代或二三代混合polish），variantCaller(Pacbio原装出品，仅可用于pacbio RSII或Sequel平台测序bam文件对基因组的polish，较慢），Racon（用于pacbio或nanopore数据对基因组polish），Pilon（用于二代数据对基因组polish）等，推荐NextPolish。

[ref.](http://blog.sciencenet.cn/blog-3406804-1205761.html)

#### nextPolish
Pacbio，nanopore，illumina基因组数据都可以用它完成校正，单一的二代或三代，或者二+三代混合polish都可以实现。
二三代混合polish一个350Mbp基因组，耗时1.5h。

1. 下载编译
```
wget https://github.com/Nextomics/NextPolish/releases/download/v1.0.5/NextPolish.tgz
tar -vxzf NextPolish.tgz
cd NextPolish && make
```

2. 添加环境变量
```
export PATH=/path_to/NextPolish:$PATH
export PATH=/path_to/NextPolish/bin:$PATH
```
3. nextpolish运行
```
nextPolish run.cfg
```

配置文件run.cfg格式如下：
```
[General]
job_type = local #运行环境，可选local,sge,pbs
job_prefix = nextPolish #作业名称
task = 1212 #迭代运行次数，12表示1轮，1212表示2轮，121212表示3轮，一般2轮就ok，提高次数可提升基因组碱基质量，但增加运行时间
rewrite = yes #是否覆盖原结果路径
rerun = 3 #重新运行未完成的作业，直到完成或达到3次循环
parallel_jobs = 1 #并行运行任务数
multithread_jobs = 5 #每个任务占用线程数
genome = /path_to/genome.fasta #组装好的基因组文件
genome_size = auto #基因组大小
workdir = ./2020_nextpolish #工作路径
polish_options = -p 4 {multithread_jobs} #polish环节参数，-p 4指4线程运行

[sgs_option] #使用二代数据polish的设置
sgs_fofn = ./sgs.fofn #指定文本文件sgs.fofn，文件写入二代rawdata（fastq或fastq.gz）文件的路径，多文件时一个路径占一行，可用命令realpath data_1.fastq data_2.fastq >sgs.fofn生成sgs.fofn文件
sgs_options = -max_depth 100 #二代polish参数设置

[lgs_option] #使用三代数据polish的设置
lgs_fofn = ./lgs.fofn #指定文本文件lgs.fofn，写入三代rawdata文件（fastq或fastq.gz)路径，多文件时一个路径占一行，可用命令realpath data_1.fastq data_2.fastq >lgs.fofn生成lgs.fofn文件
lgs_options = -min_read_len 10k -max_read_len 150k -max_depth 60 #三代reads对齐到基因组环节的参数，通过minimap2实现
lgs_minimap2_options = -x map-pb #map-pb是pacbio数据，nanopore数据用map-ont
```

nextpolish运行结束后，结果可见:
03.kmer_count/05.polish.ref.sh.work/polish_genome0/genome.nextpolish.part000.fasta


#### variantCaller

#### Racon
测序数据:sample.fasta；基因组文件:genome.fasta
 
1. 三代测序 reads 与基因组的对齐，这里使用 minimap2 来完成
对于 PacBio reads
    minimap2 -ax map-pb -t 8 genome.fasta sample.fasta > sample.sam
对于 Nanopore reads
    minimap2 -ax map-ont -t 8 genome.fasta sample.fasta > sample.sam
 
2. racon polish
    racon -m 8 -x -6 -g -8 -w 500 -t 4 reads.fastq.gz mapping.sam genome.fasta > racon1.fasta
对于获得的“racon1.fasta”，可以再拿原始reads和它对齐，再执行polish，反复2-3轮差不多就可以了。

#### Pilon

### HiC挂载
利用在组装的基因组的contigs和scaffolds的3D空间邻近关系来锚定(anchor)，排序(order),和定向(orient)这些contigs和scaffolds，已达到染色体水平基因组组装的目的。

#### 常用HiC挂载软件
1. ALLHiC：发表在Nature Plants, 用于解决高杂植物和多倍体组装问题
张兴坦老师专为多倍体和高杂合度物种基因组挂载开发。如果是复杂基因组，肯定是首选。对于简单基因组，我跑了下，结果不佳。提了issue，张老师特意开发了个为简单基因组设计的流程：https://github.com/tangerzhang/ALLHiC/blob/master/bin/ALLHiC_pip.sh，主要增加了对contig的纠错。至于效果，我还在跑。

2. 3D-DNA：发表Science，更新中
优秀的纠错功能。我认为既是优点，也是缺点。它会把你原来完整的contig拆的稀碎，认为那些不准确，需要通过染色质交互来矫正。得到的结果也是五花八门，占的空间太大了！又不敢轻易删掉，因为有些文件你在手工纠错后还要用到。
默认迭代纠错2次，根据我的折腾，你最好还是0.hic、1.hc和2.hc都试下吧，导入juice_box看下效果，哪个好就用哪个。我同时组装了两个基因组，一个是0.hic最好，另一个是1.hc最好。这个软件就很玄学，用不同的结果可能错误率差别很大。

3. SALSA2：发表在BMC genomics，更新中。
使用简单，精确度高（比3d-dna）。但存在聚类错误，调整难度大。最早提出利用Hi-C对contig进行纠错的软件，第一个使用组装软件输出的GFA信息的工具。

4. LACHESIS：2013年发表在Nature Biotechnology，2017年后无更新
经典软件，有效聚类和排序，现在发表的大部分HiC挂载文章都出自于它。但不适合多倍体和高杂合度的基因组，2017年就不再更新。
因为很旧，安装过程非常痛苦，源码安装，samtools和boost版本都要求很老。费了很大的功夫安装成功了，运行过程却总是出现：Segmentation fault (core dumped)，作者在GitHub issue上提供了解决方法（ubuntu），但对我不适用。最后放弃，建议大家也不要再用了。

[3D-DNA]（https://github.com/aidenlab/3d-dna）用得最多；还有LACHESIS，SALSA2，HiRise等也是组装Hic的软件。
[ALLHiC](https://www.nature.com/articles/s41477-019-0487-8)，唐海宝lab开发，多倍体基因组组装。

参考[Rick's pipeline](https://bioinformaticsworkbook.org/dataAnalysis/GenomeAssembly/Hybrid/Juicer_Juicebox_3dDNA_pipeline.html#gsc.tab=0)

#### ALLHiC
ALLHiC目的是为了解决基因组HiC辅助组装时，多倍体和杂合度高的基因组难以组装良好的问题。
[xuzhougeng's tutorial](http://xuzhougeng.top/archives/Assemble-genome-using-ALLHiC-with-HiC-Data)
##### ALLHiC 安装
1. 依赖
ALLHiC依赖于samtools(v1.9), bedtools 和 Python 3环境的matplotlib(v2.0+)，这些可以通过conda一步搞定。

`conda create -y -n allhic python=3.7 samtools bedtools matplotlib`

2. 安装
安装简单
```
git clone https://github.com/tangerzhang/ALLHiC
cd ALLHiC
mv allhic.v0.9.8 bin/allhic
chmod +x bin/*
chmod +x scripts/*
```

再把ALLHiC/bin和ALLHiC/scripts添加到环境变量即可使用。

3. 检查
检查下是否成功安装
`allhic -v`,`ALLHiC_prune`

可能会遇到如下的报错
`ALLHiC_prune: /lib64/libstdc++.so.6: version 'GLIBCXX_3.4.21' not found (required by ALLHiC_prune)`

是GLIBC过低导致，但是不要尝试动手去升级GLIBC（你承担不起后果的），conda提供了一个比较新的动态库，因此可以通过如下方法来解决问题
`export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/opt/miniconda3/lib`

##### ALLHiC 使用
###### 简单基因组
当基因组简单时，可用专门用于简单基因组组装的脚本ALLHiC/bin/ALLHiC_pip.sh。
`nohup ALLHiC_pip.sh -r draft.fa -1 reads.R1.fastq.gz -2 reads.R2.fastq.gz -k 12 -e HindIII -t 24 &`
参数简单。-r指定组装的基因组草图，-1和-2指定HiC测序数据，-k指定分组数量（即染色体数目），-e指定酶，-t指定线程。

###### 复杂基因组

1. 输入文件
有4个，contig序列，等位contig信息和两个双端测序数据

- Allele.ctg.table
- draft.asm.fasta
- reads_R1.fastq.gz
- reads_R2.fastq.gz

2. 建立索引
`samtools faidx draft.asm.fasta `
`bwa index -a bwtsw draft.asm.fasta`

3. 序列回贴。这一步的限速步骤是bwa sampe，因为它没有多线程参数。如果数据量很大，可以先将原始的fastq数据进行拆分，分别比对后分别执行bwa sampe，最后合并成单个文件。
`bwa aln -t 24 draft.asm.fasta reads_R1.fastq.gz > reads_R1.sai`
`bwa aln -t 24 draft.asm.fasta reads_R2.fastq.gz > reads_R2.sai` 
`bwa sampe draft.asm.fasta reads_R1.sai reads_R2.sai reads_R1.fastq.gz reads_R2.fastq.gz > sample.bwa_aln.sam`

4. SAM预处理，移除冗余和低质量信号，提高处理效率
`PreprocessSAMs.pl sample.bwa_aln.sam draft.asm.fasta MBOI`

如果已有BAM文件.
`PreprocessSAMs.pl sample.bwa_aln.bam draft.asm.fasta MBOI`
`filterBAM_forHiC.pl sample.bwa_aln.REduced.paired_only.bam sample.clean.sam`
`samtools view -bt draft.asm.fasta.fai sample.clean.sam > sample.clean.bam`
其中filterBAM_forHiC.pl的过滤标准是比对质量高于30(MQ), 只保留唯一比对(XT:A:U), 编辑距离(NM)低于5, 错误匹配低于(XM)4, 不能有超过2个的gap(XO,XG)

5. (可选): 对于多倍体或者是高杂合的基因组，因为等位基因的序列相似性高，那么很有可能会在不同套基因组间出现假信号，因此需要构建Allele.ctg.table, 用于过滤这种假信号。
`ALLHiC_prune -i Allele.ctg.table -b sample.clean.bam -r draft.asm.fasta`
这一步生成prunning.bam用于后续分析

6. 这一步是根据HiC信号将不同的contig进行分组，分组数目由-k控制。如果跳过了第四步，那么可以直接用第三步的结果sample.clean.bam
`ALLHiC_partition -b prunning.bam -r draft.asm.fasta -e AAGCTT -k 16`
这一步会生成一系列以prunning开头的文件

分组信息: prunning.clusters.txt
每个分组对应的contig: prunning.counts_AAGCTT.XXgYY.txt:
每个contig长度和count数: prunning.counts_AAGCTT.txt

7. 将未锚定的contig分配已有的分组中。
`ALLHiC_rescue -b sample.clean.bam -r draft.asm.fasta -c prunning.clusters.txt -i prunning.counts_AAGCTT.txt`
这一步根据之前prunning.counts_AAGCTT.XXgYY.txt对应的groupYY.txt

8. 优化每一组中的contig的排序和方向
生成.clm文件
`allhic extract sample.clean.bam draft.asm.fasta --RE AAGCTT`

优化

```
for i in group*.txt; do
    allhic optimize $i sample.clean.clm
done
```

这一步会基于groupYY.txt生成对应的groupYY.tour

9. 将tour格式转成fasta格式，并生成对应的agp。
`ALLHiC_build draft.asm.fasta`
这一步生成两个文件，groups.asm.fasta和groups.agp。其中groups.asm.fasta就是我们需要的结果。

10. 构建染色质交互矩阵，根据热图评估结果
`samtools faidx groups.asm.fasta`
`cut -f 1,2 groups.asm.fasta.fai  > chrn.list`
`ALLHiC_plot sample.clean.bam groups.agp chrn.list 500k pdf`

11. 使用ALLHiC的几个注意事项:

- ALLHiC依赖于初始的contig，如果嵌合序列和坍缩序列比例过高，那么ALLHiC结果也会不准确。根据文章，ALLHiC能够处理~10%的嵌合比例，~20%的坍缩比例。因此最好是用类似于Canu这种能够区分单倍型的组装软件。
- 单倍型之间序列相似度不能太高，否则会出现大量非唯一比对，降低可用的HiC信号
- 构建Allele.ctg.table需要一个比较近缘的高质量基因组
- 不要用过短的contig，因为短的contig信号少，很容易放到错误的区域
- K值的设置要根据实际的基因组数目设置，如果你发现输出结果中某些group过大，可以适当增大k值


#### HiC Pipeline-Juicer+3D-DNA+Juicebox
[aidenlab's official tutorail](http://aidenlab.org/assembly/manual_180322.pdf)
[xuzhougeng's tutorial](http://xuzhougeng.top/archives/scaffolding-genome-using-3d-dna-workflow)


##### 软件安装
1. 依赖安装
2. juicer安装（CPU版本）
```
git clone https://github.com/theaidenlab/juicer.git
cd juicer
ln -s CPU scripts
cd scripts/common
wget https://hicfiles.tc4ga.com/public/juicer/juicer_tools.1.9.9_jcuda.0.8.jar
ln -s juicer_tools.1.9.9_jcuda.0.8.jar  juicer_tools.jar
```
用`juicer/scripts/juicer.sh -h`检查安装。可能需要chmod 771 3d-dna/run*修改两个sh文件为可执行。

3. 3D-DNA安装
`git clone https://github.com/theaidenlab/3d-dna.git`
用`3d-dna/run-asm-pipeline.sh -h`检查安装。可能需要chmod 771 3d-dna/run*修改两个sh文件为可执行。

##### 软件运行
###### juicer
Juicer transforms raw sequence data into a list of Hi-C contacts (pairs of genomic positions that were adjacent to each other in 3D space during the experiemnt).
Juicer把Hi-C测序的原始序列数据转化成成对的基因组相邻位置的Hi-C关联信息列表。

软件运行时间基于300Mbp基因组估计。

1. 准备工作
在工作目录下创建fastq和reference，分别存放HiC序列和组装好的基因组。
一定要是这两个名称，juicer特异性识别fastq（以及fastq下的_R*.fastq*文件名），3d-dna特异性识别reference。

建立参考基因组的索引，生成同名的.amb .ann .bwt .pac .sa文件。建议在运行目录下创新reference目录存放基因组和相关文件。
输入基因组序列：需要80个碱基一行的格式，如果不是用`seqkit seq -w 80 before.genome.fa > after.genome.fa`获取。

`bwa index genome.fa && echo "**bwa index done **"`

time：10min。

2. 根据基因组构建可能的酶切位点位置文件
`python ./juicer/misc/generate_site_positions.py DpnII sample genome.fa`
生成genome_DpnII.txt文件

第一个参数选择酶切类型：
HindIII(AAGCTAGCTT)(六碱基识别序列，百迈客使用), MboI(GATCGATC)（四碱基识别序列）, DpnII(GATCGATC)（四碱基识别序列，诺禾和贝瑞使用）, NcoI(CCATGCATGG)；序列的名称里会包含HindIII(TAGCTT),DpnII(GA+TC)等特征字符可以粗略判断。其中DpnI, DpnII, MboI, Sau3AI, 识别相同的序列（GATC），仅仅是对甲基化敏感度不同。
第二个参数是样品名称：自行设置。
第三个参数输入基因组序列。

time：3min。

3. 获取每条contig的长度
`awk 'BEGIN{OFS="\t"}{print $1, $NF}' sample_DpnII.txt > sample.chrom.sizes`

4. 运行juicer（CPU版本）
在当前目录下创建存放HiC序列文件的目录`mkdir fastq`（一定要是fastq这个名称，juicer软件只识别这个名称），再把HiC序列放进fastq目录，也可以创建软连接来替代`ln -s /path to HiC fastq/sample_R1.fastq.gz ./fastq/sample_R1.fastq.gz && ln -s /path to HiC fastq/sample_R2.fastq.gz ./fastq/sample_R2.fastq.gz`（一定要是_R*.fastq*这样的格式，还是juicer的特异性识别）。

`nohup time bash /soft/juicer/scripts/juicer.sh -g sample -d ./ -s DpnII -p sample.chrom.size -y sample_DpnII.txt -z genome.fa -D /soft/juicer -t 36 >juicer.log 2>&1 &`
HiC数据必须以格式`fastq/*_R*.fastq*`存放在运行目录下的fastq目录中，否则报错;
-p,-y,-z三个参数是必须设置的；其余建议设置;
运行时间较长，建议加上`nohup time command >juicer.log 2>&1 &`实现计时和后台运行。

-g genomeID；可用sample样品名。
-d topDir;设置运行目录，会调用运行目录下的fastq目录内的fastq文件；并在运行目录下生成splits目录存放临时文件，aligned目录存放结果文件；默认是当前目录。
-s 酶切类型；HindIII(AAGCTAGCTT)(百迈客使用), MboI(GATCGATC) , DpnII(GATCGATC)（诺禾和贝瑞使用）, NcoI(CCATGCATGG)；序列的名称里会包含HindIII(TAGCTT),DpnII(GA+TC)等特征字符可以粗略判断。其中DpnI, DpnII, MboI, Sau3AI, 识别相同的序列（GATC），仅仅是对甲基化敏感度不同，可以通用。
-S stage；定义重新运行的阶段；可断点重跑部分程序。
-p sample.chrom.size；前面步骤获得的contig长度文件。
-y sample_DpnII.txt；前面步骤获得的限制性酶切位点位置文件。
-z genome.fa；基因组文件。
-D juicer scripts directory; juicer程序目录，目录下有存放命令文件的scripts目录。
-t threads；线程。
-b ligation。
-a 实验描述说明；可以不设置。

生成的结果文件在aligned目录下，下一步3d-dna要用的是aligned/merged_nodups.txt，保存着去除重复的，成对的基因组相邻位置的Hi-C关联信息列表。

time：300Mb的基因组12个线程运行了11hours。

###### 3d-dna
3d-dna基于Hi-C数据，来纠正错误的组装，锚定(anchor)，排序(order),和定向(orient)组装的draft的contigs和scaffolds代表的DNA片段。
3d-dna的工作流程包括：
- 对输入的draft genome进行scaffoldding
- 寻找scaffolded的序列的mismatches，并列出不一致的区域
- 根据不一致区域重新编辑输入的draft genome文件
- 再次进行scaffoldding
- 多轮mismatches check【optional】之后
- 进行polish
- 对polished megascaffold进行split分割，生成raw chromosomal scaffolds
- 对分割后的scaffolds进行seal封装
- 对封装好的sealed scaffols进行merge合并【optional】
- 根据合并好的chromo-some-length-scaffolds染色体长度的scaffolds生成最终的final fasta基因组文件

`nohup run-asm-pipeline.sh -r 2 reference/M.candidum.fa aligned/merged_nodups.txt > 3d.log 2>&1 &`
输入参数：
-i input_size: 长度低于给定阈值的contig/scaffold被过滤掉, 默认是15000。
-r number_of_edit_rounds: 基因组中misjoin的纠错轮数，默认是2；当基因组比较准确时，设置为0，然后在JABT中调整会更好。(设置成2时生成的*.0.hic和*.0.assembly这套结果和设置成0时一样)
-m haploid/diploid: 单倍体模型还是二倍体模型，默认是单倍体haploid；组装的单倍型基因组明显偏大（表明杂合度较高）时使用二倍体模型，会调用merge模块。
-s stage: 从polish, split, seal, merge 或finalize 的某一个阶段开始运行。

输出文件：
.fasta: 以FINAL标记的是最终结果
.hic: 多分辨率的关联矩阵contact matrices，各个阶段（包括纠错的所有rounds,sealed,polished,resolved,final等阶段）都会有输出结果，用于在JABT中展示。
.assembly: 一共两列，存放对draft sequences的一套操作（包括分割splitting，改变顺序changing order，定向orientation和锚定到染色体anchoring into chromosomes）的简明指令。各个阶段都会有输出，与.hic对应。
.scaffold_track.txt & .superscaf_track.txt：以Juicebox 2D注释格式保存的单独的scaffold和superscaffold（chromosome）的边界boundary文件。可用于circos作图的输入。
.bed & .wig: 1D-轨迹文件，解释用于错误连接探测misjoin detector，重复探测collapsed repeat detector，抛光polisher和染色体分割chromosome slitter的指令。一般没啥用。
edits.for.step.\*.txt;mismatches.at.step.\*.txt;suspect_2D.at.sept.\*.txt: 用juicebox 2D注释格式列出有问题区域。
alignments.txt(diploid mode才有的)：LASTZ生成的候选单倍型的成对alignment数据。

设置的-r 2会迭代0-2轮，生成0，1，2和final后缀的四套*.hic和*.assembly文件，都可用于后续juicebox的调整。
可以每一套都导入juicebox看看，选择效果最好的一套用于juicebox手动调整。（常常0后缀的这套有更好的效果，0后缀这套的assembly文件的所有scaffolds都属于一个chrome，需要自己手动切割chromes）

time：5.5hours。

###### visualize candidate assembly
如果对组装的基因组很有信心，或者是已经到染色体水平的基因组（有时3d-dna纠错之后反而contigs数量增多和变碎，难以用juicebox调整也可以这样操作），可以不运行3d-dna的run-asm-pipeline.sh纠错脚本，而直接用下面命令生成genome.assembly和genome.hic用于juicebox的手动调整。
awk –f ./3d-dna/utils/generate-assembly-file-from-fasta.awk genome.fa >
genome.assembly
./3d-dna/visualize/run-assembly-visualizer.sh genome.assembly ./aligned/merged_nodups.txt

###### juicebox手工纠错
- juicebox软件安装
在window系统下，在[aidenlab software](https://aidenlab.org/software.html)页面，或者[juicebox github](https://github.com/aidenlab/Juicebox/wiki/Download)下载最新版本的可执行文件，比如[juicerbox](https://s3.amazonaws.com/hicfiles.tc4ga.com/public/Juicebox/Juicebox_1.11.08.exe)，可直接打开使用。

- juicebox操作
参考[Aiden Lab官方制作的juicerbox教程](https://www.youtube.com/watch?v=Nj7RhQZHM18)和
[juicerbox教程中文翻译](https://www.bilibili.com/video/BV1LK411M7nZ?from=search&seid=15631142475797820614)

输入文件：从3d-dna获得的\*\*final.hic文件（热图）和\*\*final.assemble文件（contig位置信息，方框）；
注意大小写，有final和FINAL文件同时存在。

打开juicerbox，File-open-local选择\*\*final.hic文件，可以获得一个热图，显示任意一对位点之间的接触频率，即物理距离，颜色越深物理距离越近；再Assembly-Import Map Assembly选择\*\*final.assemble文件，右下角载入了3层注解，绿色-Scaf代表Scaffolds，蓝色-Chr代表染色体，黄色edit代表选中的区域，出现一些方框，表示了这个contig与热图对应的位置。

期望正确连接的热图是在左上到右下对角线位置附近形成一条颜色深的红带，而其他位置颜色浅或无颜色(染色体内部的物理距离明显高于染色体之间）。在juicerbox上做的是调整序列在对角线上的错误位置和顺序，以实现正确地体现染色体包含的scaffolds和scaffolds的顺序。

常见的几种组装错误:

|常见组装调整|热图表现|处理方法|具体操作|
|---|---|---|---|
|misjoin-错误连接|一个染色体内的一个scaffold内的某一点左下和右上几乎空白（颜色很浅），表示这个点连接的两段序列的物理距离很远，而错误的被放在一起连接。|把错误连接的scaffold的其中一段序列剪切，放到最后，等待进一步处理（比如根据其他组装错误放在其他染色体的位置）|选择：按住shift键，鼠标点选这个错误的scaffold，出现一对黄黑色平行横线和一对黄黑色平行纵线，表示框内sacaffold被选中（按住shift，单击鼠标可以取消选择）。剪切：选择之后，鼠标放上去会出现剪刀符号和一个白框，单击则会剪切白框内的序列放到整个assembly的最后，属于一个单独的染色体，并且在scaffold内，剪切序列前后的上下游序列各自形成新的scaffold。为了更精准，可以调整Resolution放大视图后再进行剪切；滚动鼠标可以调整白框大小，改变剪切的范围大小。|
|translocation-易位|在对角线以外的区域出现水平和垂直的深色条带，代表本来物理距离近的序列被放在了远的位置|调整这段序列到水平和垂直的深色条带颜色最深的交叉位置|选择：操作同上。插入：鼠标移动到未被选中的区域以外，两个scaffolds之间的位置时，出现箭头，点击会把选中的区域移动到箭头的位置。移动之后，远离对角线的水平和垂直深色条带消失代表移动纠正了错误。|
|inversions-倒置|一个scaffold附近出现领结状热图，代表scaffold的末端离上游的物理距离更近，首端离下游的物理距离更近，这个scaffold被错误地倒置了。|翻转这个scaffold|选择这个scaffold。reverse-翻转：鼠标放在选中的scaffold的右上角或左下个角上，出现圆圈箭头提示符，单击即可翻转选中的scaffold，热图上马上可以看到纠正后效果。|
|chromosome boundaries-染色体边界|热图会在不同染色体间出现明显间断|根据热图间断确定染色体边界|在没有选中任何区域的状态下，鼠标放在两个染色体或者scaffolds的间断点附近时，出现方形的一半符号（7字符或L字符），此时单击会分割染色体（两个scaffolds间）或者合并染色体（两个染色体间）。另外，选中单个或多个scaffolds或者染色体，右键-add chr boundaries可以为所有scaffolds添加染色体边界；右键-remove chr boundaries为合并所有选中染色体的边界。|

常用操作：
- 放大缩小窗口：选择可以显示颜色的最小resolution(BP)后，点击resolution右边的锁定符号锁定显示分辨率。若是某锁定状态，双击是改变resolution，若是锁定状态，双击则只是窗口放大。一般锁定最好的分辨率后，用双击来放大指定区域，用右键undo zoom和redo zoom来持续放大缩小指定区域。

右键后的一些操作/选项：
- undo：撤销，可以快捷键ctrl+U；redo：重做，可以快捷键ctrl+R。
- undo zoom：撤销zoom窗口；redo zoom：重新zoom窗口。
- move to debris：把选中的一个或多个scaffolds放在不属于已有染色体的末尾碎片区域。与misjoin的剪刀剪切的区别是，misjoin的剪刀剪切是分割一个scaffold内部，成为多个scaffolds；move to debris是对scaffold整体操作，通常用于信号很弱的大块区域。



如果查看不同的*.hic和*.assembly文件，可以同时打开多个juicebox窗口，使用File-New Window打开新的窗口，不要在已有打开文件基础上再次打开。

检查并纠正所有错误后即可输出assembly文件。Assembly-export assembly输出genome.review.assembly用于下一步的分析。
可以用附带的命令行工具基于genome.review.assembly生成最终的fasta序列。


推荐调整操作的步骤：
- 使用0.hic和0.assembly导入juicebox。
- 根据热图手动指定染色体边界，尽量与物种染色体数量一致。如果有跨越染色体的scaffold，用剪刀剪开后再指定染色体边界；如果有信号很弱的区域，用右键-move to debris移动到末端。
- 接着在染色体内部，对染色体依次操作。
- 依次把染色体内明显是错误连接misjoin的scaffold剪开，以便操作。
- 依次把染色体间和染色体内易位translocation的区域放回正确的位置。
- 在染色体内部依次检查倒置inversions。
- 从头到尾依次再一次检查错误连接misjoin，染色体内易位translocation，倒置inversions和染色体边界chromosome boundaries。


###### rerun-3d-dna
根据juicerbox手工纠正的结果, genome.review.assembly, 使用run-asm-pipeline-post-review.sh重新组装基因组。
`run-asm-pipeline-post-review.sh -r genome.review.assembly genome.fa aligned/merged_nodups.txt &`

### 组装结果评估
#### busco
[blog_busco](https://yanzhongsino.github.io/2021/07/24/biosoft_busco/)

#### MUMmer
比较不同组装软件的结果，评估polish基因组前后差异
用MUMmer共线性分析比较不同组装软件获得的基因组

mkdir mummer && cd mummer

ln -s ./mecat.fasta necat_cut.fasta
ln -s ../genome/nextdenovo_cut.fasta nextdenovo_cut.fasta
ln -s ../nextdenovo/03.kmer_count/05.polish.ref.sh.work/polish_genome0/genome.nextpolish.part000.fasta nextdenovo_cut_polish.fasta
ln -s ../necat/03.kmer_count/05.polish.ref.sh.work/polish_genome0/genome.nextpolish.part000.fasta necat_cut_polish.fasta
 
#polish 前的基因组，NextDenovo 和 NECAT 两个软件的组装结果
nucmer --mum -p raw nextdenovo_cut.fasta necat_cut.fasta
delta-filter -m raw.delta > raw.filter
show-coords -T -r -l raw.filter > raw.coords
mummerplot --postscript -p raw raw.filter
ps2pdf raw.ps raw.pdf
 
#polish 后的基因组，NextDenovo 和 NECAT 两个软件的组装结果
nucmer --mum -p nextpolish nextdenovo_cut_polish.fasta necat_cut_polish.fasta
delta-filter -m nextpolish.delta > nextpolish.filter
show-coords -T -r -l nextpolish.filter > nextpolish.coords
mummerplot --postscript -p nextpolish nextpolish.filter
ps2pdf nextpolish.ps nextpolish.pdf

#可分别查看两个 *.coords 或 *.pdf
#### QUAST
[QUAST(Quality Assessment Tool for Genome Assemblies)](https://github.com/ablab/quast)可以统计组装的genome的基本信息，包括序列长度，N50，L50,GC含量等。

QUAST的在线使用：[QUAST在线网站](http://cab.cc.spbu.ru/quast/)，限制上传序列在100Mb以内，可选参数较少。
在线网站使用方法：上传序列，选择参数，填写邮箱，然后点击Evaluate，等待邮箱收取运行结果。

QUAST的本地安装：`conda install quast`或者下载解压`wget -c https://github.com/ablab/quast/releases/download/quast_5.1.0rc1/quast-5.1.0rc1.tar.gz && tar -zxvf quast-5.1.0rc1.tar.gz`即可使用。

QUAST的本地使用：
1. 无参评估：用于统计基因组序列长度，GC含量，N50等基本信息。
   `quast.py sample.fa -t 4 -o ./outdir/`
   -t线程，-o输出目录；
   结果主要看icarus.html网页报告，比较全面。点击“icarus.html”后，在目录页点击“QUAST report”，即可查看评估结果的基本统计内容。新界面（即链接至结果文件夹中的“report.html”）中的主要内容包括了组装结果的基本信息，如拼接后的序列总数、序列长度、GC含量、N50、长度累积曲线、GC含量曲线等。
   返回报告“icarus.html”中点击“Contig size viewer”，或者在以上界面（即“report.html”）中直接点击左上方的“View in Icarus contig browser”后，则可在新界面中以一个简单的基因组浏览器的形式，查看基因组序列组成（即基因组序列由哪些contigs/scaffolds组成）、长度、N50等信息。该界面中可自定义展示区间长度，可通过在“start”或“end”中输入特定区间数值，也可在图中拖动黄色区块查看。
2. 有参评估：添加已有参考基因组的信息，结果更全面，包括与参考基因组的对比。
   `quast.py sample.fa -r ref.fa -g ref.gff --fragmented -t 4 -o ./outdir/`
	--fragmented: detect misassemblies caused and mark them fake；
	“Contig size viewer”界面还根据组装结果与参考基因组的align信息，在该组装结果的展示图中将组装结果与参考基因组之间的一致区域、非一致区域等标记出来，以更好地帮助我们对组装基因组结果进行评估。
	备注：对于每一条scaffold/contig，只要存在很少一部分与参考基因组不一致的区域，即将整条scaffold/contig判定为“misassembled contigs”（即错误装配的contigs，在图中标记为红色区域）。

QUAST的结果
结果文件


report.txt的范例
```
Assembly                    sample
# contigs (>= 0 bp)         266
# contigs (>= 1000 bp)      266
# contigs (>= 5000 bp)      186
# contigs (>= 10000 bp)     159
# contigs (>= 25000 bp)     108
# contigs (>= 50000 bp)     30
Total length (>= 0 bp)      256218469
Total length (>= 1000 bp)   256218469
Total length (>= 5000 bp)   256043090
Total length (>= 10000 bp)  255847102
Total length (>= 25000 bp)  254902657
Total length (>= 50000 bp)  252626053
# contigs                   266
Largest contig              33924140
Total length                256218469
GC (%)                      42.94
N50                         20460156
N90                         13725599
L50                         5
L90                         11
# N's per 100 kbp           66.54
```


### genome annotation
repeatmasker

repeatmodeler

[geta](https://github.com/chenlianfu/geta)做第一轮注释
准备如下数据

Pfam数据库：Pfam33.1版本的Pfam-A.hmm.gz文件（ftp协议），下载之后gzip -d Pfam-A.hmm.gz解压，hmm是文本文件，用hmmpress Pfam-A.hmm对进行二进制转化（加快运算）并压缩和建立数据库索引，生成四个Pfam-A.hmm前缀文件。
wget ftp://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam33.1/Pfam-A.hmm.gz
gzip -d Pfam-A.hmm.gz
hmmpress Pfam-A.hmm

Unipro蛋白数据库：
wget ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/uniprot_sprot_plants.dat.gz
zcat uniprot_sprot_plants.dat.gz |\
    awk '{if (/^ /) {gsub(/ /, ""); print} else if (/^AC/) print ">" $2}' |\
    sed 's/;$//'> uniprot_sprot_plants.fa


wget ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz
gzip -d uniprot_sprot.fasta.gz




wget ftp://ftp.ncbi.nih.gov/pub/UniVec/UniVec -O UniVec.fasta
formatdb -i UniVec.fasta -p F -o T