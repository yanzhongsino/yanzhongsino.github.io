---
title: 基因组注释（一）：重复序列注释
date: 2021-08-02 10:30:00
categories: 
- omics
- genome
- annotation
tags:
- genome
- genome annotation
- repeat sequences
- transposable elements(TE)
- RepeatMasker
- RepeatModeler
- EDTA
- tandem repeat

description: 记录了基因组注释的第一步，重复序列的注释，转座子的相关知识，注释常用软件和用法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=365639&auto=1&height=32"></iframe></div>

**防坑指南**
1. 基因组在组装后，注释前。`seqkit seq -u lower.geno >upper.geno`改为碱基全部大写的形式，因为小写碱基有些软件是识别成soft-masked的形式，影响注释预测，edta的TIR预测和geta软件都有可能报错。（这个坑我已经踩两次了，唉~）

**写在前面**

多种参数和条件用下来推荐的方案是：
1. RepeatModeler自我训练
2. RepeatMasker加载Repbase数据库，并用-species参数指定近缘种，参考RepeatModeler自我训练结果做重复序列预测
3. EDTA，提供转录组组装的CDS序列做过滤，实现Denovo的TE预测。
4. 结合RepeatMasker和EDTA的结果作为最终基因组的重复序列注释。

# 1. 重复序列注释
重复序列占基因组非常高的比例，对重复序列的注释一般是做基因组注释的第一步。常用的基因组重复序列注释软件有RepeatMasker, RepeatModeler, EDTA。

转座子(transposable elements,TE)是可以在基因组内改变位置的一段DNA序列，通常由DNA复制造成，TE是基因组的重要组成部分。

<table>
	<caption><h4>转座子(transposable elements,TEs)的分类</h4></caption>
	<thead>
	<tr>
		<th>转座子</br>(transposable elements,TEs)</th>
		<th>功能</th>
		<th colspan="2">分类</th>
		<th>结构特征</th>
	</tr>
	</thead>
	<tbody>
	<tr>
		<td rowspan="5"><h6>I 类元件</br>/逆转录转座子(retrotransposons)</h6></td>
		<td rowspan="5">在“复制和粘贴”转座机制中作为RNA中间体</td>
		<td rowspan="3">长末端重复(long terminal repeat, LTR)逆转录转座子</td>
		<td>Ty3-gypsy类</td>
		<td>LTR 元件具有 5 bp 目标位点重复(TSD)；LTR-Gypsy元件为 5'-TG...CA-3'。</td>
	</tr>
	<tr>
		<td>Ty1-copia类</td>
		<td>LTR 元件具有 5 bp 目标位点重复(TSD)；LTR-Copia元件的标准末端序列为 5'-TG...C/G/TA-3'</td>
	</tr>
	<tr>
		<td>其他，比如BEL-Pap类，内源性反转录病毒(HERV,MER4)等</td>
		<td>---</td>
	</tr>
	<tr>
		<td rowspan="2">非长末端重复(non-LTR)逆转录转座子</td>
		<td>长散布核元件</br>(LINEs,long interspersed nuclear element)</td>
		<td rowspan="2">non-LTR 具有可变长度的 TSD 或完全缺乏 TSD，而是与插入时侧翼序列的缺失相关。non-LTR 通常在元件的 3' 末端有一个末端 poly-A 尾</td>
	</tr>
	<tr>
		<td>短散布核元件</br>(SINEs,short interspersed nuclear element)</td>
	</tr>
	<tr>
		<td rowspan="7"><h6>II 类元件</br>/DNA 转座子(DNA transposons)</h6></td>
		<td rowspan="7">在“剪切和粘贴”转座机制中作为DNA中间体</td>
		<td rowspan="5">Terminal Inverted Repeat(TIR)元件</br>(TIR末端反向重复序列,这里指具有TIRs结构的DNA转座子，包括微型反向转座元件MITEs)</td>
		<td>CACTA</td>
		<td>CACTA元素被相对较短的TIRs(15-100bp)识别，这些TIRs以保守的CACTA...TAGTG基序终止，有3bp TSD。</td>
	</tr>
	<tr>
		<td>Mutator</td>
		<td>两侧通常有非常大的反向重复序列inverted repeats(200-500bp)，两侧有9bp的目标位点重复。</td>
	</tr>
	<tr>
		<td>PIF/Harbinger</td>
		<td>包含通常以一小段C/G结束，两侧有富含T/A的3bpTSD的短TIRs。许多是具有共同的富含G的TIR序列的旅游团(Tourist Group)的MITEs</td>
	</tr>
	<tr>
		<td>Tc1/Mariner</td>
		<td>包含长度约10-30bp，通常两侧有TA的TSD的短TIRs。</td>
	</tr>
	<tr>
		<td>hAT</td>
		<td>只有极少数被分类在TREP。特征还不好描述，TIR可以很短(几个bp)，两侧有一个8bp的TSD。</td>
	</tr>
	<tr>
		<td colspan="2">Helitron</td>
		<td>通过滚环机制复制，因此不产生 TSD 序列，也没有 TIR，但具有特征 5'-TC…CTRR-3' 末端序列，通常在元素的 3' 端附近有一个富含 GC 的短茎环结构。</td>
	</tr>
	<tr>
		<td colspan="2">Mavericks</br>非TE重复序列</br>(non-TE repeat sequence)</td>
		<td>大尺寸，15-40 kbp</br>包含长的TIRs(几百对碱基)</br>包含保守的5'-AG...TC-3'末端</td>
	</tr>
	<tr>
		<td rowspan="3">串联重复(tandem repeats)</td>
		<td rowspan="3">串联重复序列是一个或多个核苷酸的模式重复发生，并且重复彼此直接相邻的序列。</td>
		<td colspan="2">二核苷酸重复(dinucleotide repeat)</br>三核苷酸重复(trinucleotide repeat)</td>
		<td>根据重复单元的核苷酸数量，可以分为二核苷酸重复(dinucleotide repeat)，三核苷酸重复(trinucleotide repeat)等。</td>
	</tr>
	<tr>
		<td>小卫星(minisatellite)：</br>当重复单元的核苷酸数量为10-60时，称为小卫星，通常重复5-50次。</td>
		<td>微卫星(microsatellites)：</br>在遗传谱系或法医领域，又被称为短串联重复(short tandem repeats,STR)，在植物遗传学领域，又被称为简单序列重复(simple sequence repeat, SSR)</td>
		<td>小卫星中那些重复单元中核苷酸数量较少的被称为微卫星重复，核苷酸数量从1到6或者更多，没有统一规定。</td>
	</tr>
	<tr>
		<td colspan="2">可变数量串联重复(variable number tandem repeat, VNTR)</td>
		<td>当重复单元拷贝数在所分析的群体中是可变时，被称为可变数量串联重复(VNTR)。MeSH将VNTR分类在小卫星下。</td>
	</tr>
	</tbody>
</table>

# 2. RepeatMasker
## 2.1. RepeatMasker介绍
RepeatMasker是基因组重复序列检测的常用工具。一般依赖于已有的重复序列参考库Repbase作同源预测。对于绝大部分目标真核物种，都收录在Repbase中。

## 2.2. RepeatMasker安装
1. conda安装
`conda install -c bioconda repeatmasker`

2. 加载Repbase数据库【推荐】

- 强烈建议加载[Repbase数据库](https://www.girinst.org/server/RepBase/index.php)，遗传信息研究所发布的重复DNA数据库，收录了非常多的物种基因组的重复序列信息，目前是需要付费的，加载后重复序列注释率得到很大提升。

- 上传了一个20181026的版本到百度云盘，可以下载：
链接：https://pan.baidu.com/s/1g43z1DyHtiGxAGJMiC4nZQ 
提取码：ckc4

- 下载解压后把目录下两个文件RMRBSeqs.embl和README.RMRBSeqs放到RepeatMasker安装目录的Libraries目录下，如果是conda安装的在pathto/anaconda3/share/RepeatMasker/Libraries/下。

3. 配置依赖

- 在RepeatMasker目录下运行`perl ./configure`，依次配置trf的位置，选择搜索引擎（4选1，推荐RMBlast或者HMMER（nhmmer命令），可以配置多个，但运行时只能用一种，选完再选5）。

- 一般如果依赖的软件安装成功加入环境变量了，会在配置依赖时自动出现依赖软件的路径，如果没有则手动输入依赖软件的路径。最后提示RepeatMasker is now ready to use即表示配置成功。

## 2.3. RepeatMasker使用
### 2.3.1. 查看数据库中的物种
1. Repbase数据库配置好以后，用`RepeatMasker/util/queryRepeatDatabse.pl -tree >species.txt`可以在species.txt中查看数据库中的物种列表。
2. 查看RepeatMasker/Libraries/taxonomy.dat文件，也有所有已收录物种的名称。

- 找到数据库中已有训练的物种或者近缘种后，在使用RepeatMasker时用参数-species指定库中已有物种作为参考。

### 2.3.2. RepeatModeler自我训练
- 大部分情况下数据库中没有待注释物种的近缘种，此时最好用RepeatModeler等软件进行自我训练，用denovo预测来构建重复序列数据库。
- 即使数据库中已有近缘种，有时候Repbase注释重复区的效果不是很好，也推荐进行自我训练。

#### 2.3.2.1. RepeatModeler安装
1. RepeatModeler依赖：
- 配置好的RepeatMasker
- RECON：Denovo Repeat Finder
- RepeatScout：Denovo Repeat Finder
- NSEG

2. RepeatModeler支持conda安装：`conda install -c bioconda repeatmodeler`

3. 或者下载源码编译，再一步步指定依赖软件路径，类似RepeatMasker的安装过程。

```
wget http://repeatmasker.org/RepeatModeler/RepeatModeler-open-1.0.11.tar.gz
tar xzvf RepeatModeler-open-1.0.11.tar.gz
 
cd RepeatModeler-open-1.0.11
chmod -R 755 *
perl ./configure
```

#### 2.3.2.2. RepeatModeler使用
1. 建库

生成sample.nhr,.nin,.nnd,.nni,.nog,.nsq,.translation 7个文件

`mkdir db && cd db`
`BuildDatabase -name sample -engine ncbi sample.fa`

2. self-training

`nohup RepeatModeler -pa 8 -database db/sample -engine ncbi &`

- 耗时数天。
- -pa 限制的是core的进程，如果服务器是4核，想设置32个线程跑，应该设置-pa 8。
- -databse指定库
- -engine指定引擎
运行跑6 rounds，前4轮有几分钟到十几分钟的-pa线程×cores的超负荷运行，后两轮每轮约1-2h的-pa线程×cores的超负荷运行。

3. 结果文件
- sample-families.fa （用于repeatmasker指定lib）
- sample-families.stk（用于上传到Dfam数据库）
- 生成目录下RM_21945.SatAug11650032020，内含consensi.fa文件（自身比对找到的一致性序列）
- consensi.fa.classified文件（fa格式），包含所有repeat序列和分类，序列id里包含了序列名称和是否能被repeatmasker识别的状态，如果不能被识别就标记Unknown。

consensi.fa.classified文件即为训练结果，重复序列数据库，用作其他软件的输入。

4. 处理

如果想要更加可信的结果，可以把标记为Unknown的去除，留下的保存为ModelerID.lib，作为后续使用。

- `seqkit grep -r -i -p "Unknown" -v ./RM_21945.SatAug11650032020/consensi.fa.classified > ModelerID.lib`  #从consensi.fa.classified 文件提取可以被识别（就是序列id不含Unknown的序列）

- `seqkit grep -r -i -p "Unknown"  ./RM_21945.SatAug11650032020/consensi.fa.classified > Modelerunknown.lib`  #从consensi.fa.classified 文件提取不能被识别（就是序列id含Unknown的序列）

### 2.3.3. RepeatMasker运行
`RepeatMasker sample.fa -species "Arachis ipaensis" -pa 12 -lib consensi.fa.classified -poly -html -gff -dir repeatmasker`

**参数解释：**
- -species 选择RepeatMasker数据库中已有近缘物种，比如"Arachis ipaensis"，可用RepeatMasker/util/queryRepeatDatabase.pl -tree >species.txt获取已有物种列表。但RepeatMasker-4.1.1版本及以后不再提供queryRepeatDatabase.pl脚本，而是由famdb.py代替。
- -pa 线程，用RMBlast引擎会用到4核，即设置-pa 12会用到48线程。
- -lib RepeatModeler.consensi.fa.classified RepeatModeler自我训练的库
- -poly 在file.poly中保存多态的polymorphic的简单重复序列，即微卫星重复序列。
- -html输出xhtml格式文件
- -gff输出gff格式文件
- -dir 指定输出文件的目录，默认时当下目录

### 2.3.4. RepeatMasker结果
- sample.fa.tbl：结果报告，统计了基因组长度，GC含量，重复序列长度及各类重复序列占比等。
- sample.fa.out：将基因组中预测得到的重复序列和参考序列相比的碱基替换频率、插入/删除率，以及重复序列的位置、结构、类型等信息展示出。
- sample.fa.out.gff：sample.fa.out的gff格式。
- sample.fa.out.html：sample.fa.out的网页形式。
- sample.fa.polyout：通过-poly参数，额外将sample.fa.out中的微卫星注释提取出来。微卫星注释是一种Simple_repeat序列，可以通过*.polyout将*.out的微卫星注释删除。有研究者认为微卫星不能算作严格的重复序列类型，还有研究者甚至认为Simple_repeat全都不能算。
- sample.fa.cat.ga：记录了基因组序列和数据库中参考重复序列的比对详情，其中“i”和“v”分别代表了碱基转换（transitions）和颠换（transversions），“-”表示该位点存在碱基插入/删除。。
- sample.fa.masked：将重复序列屏蔽成N碱基的masked基因组。


其中，sample.fa.out每一列含义：
- 第一列：比对分值，SW score
- 第二列：替代率 perc div.
- 第三列：碱基缺失百分率
- 第四列：在重复序列中碱基缺失百分率
- 第五列：query sequence
- 第六列：查询序列起始位置
- 第七列：查询序列终止位置
- 第八列：查询区域中超出比对区域碱基的数- 目，也就是没有比对上的碱基数
- 第九列：+/-(C)
- 第十列：比上的重复序列名称，类型命名
- 第十一列：比上重复序列的分类，和- repeatmolder 中*.classed 是一样的
- 第十二列：比上的在数据库中的起始位置
- 第十三列：比上的在数据库中的终止位置
- 第十四列：在第十列上超出比对区域碱基的数- 目，也就是没有比对上的碱基数
- 第十五列：比对区域的ID，随机给的

# 3. EDTA(The Extensive de novo TE Annotator)
## 3.1. EDTA介绍
[EDTA](https://github.com/oushujun/EDTA#quick-installation-using-conda-linux64)是一个用于自动化全基因组的转座元件(transposable elements,TE)从头注释(de-novo)注释和评估TE注释的综合软件。

<img src="https://github.com/oushujun/EDTA/raw/master/development/EDTA%20workflow.png?raw=true" title="EDTA流程图" width="90%" />

**<p align="center">Figure 1. EDTA流程图**
from [github: EDTA](https://github.com/oushujun/EDTA#quick-installation-using-conda-linux64)</p>

**EDTA运行和调用软件的流程：**
1. LTR预测
+ LTR_FINDER预测LTR
+ LTRharvest预测LTR
+ LTR_retriever确认前两个软件预测的LTR
2. TIR预测
+ Generic Repeat Finder
+ TIR-Learner预测TIR
3. Helitrons预测
+ HelitronScanner
4. filter过滤
+ 以上三种TE预测结束后做basic filter获得full-length TEs
+ 进一步做advance filter，获得raw library
+ 与指定的curated library合并
+ 用RepeatModeler做训练
+ 用指定的CDS做最后的filters
5. 获得最终的final TE library
6. Annotation
7. Evaluation

## 3.2. EDTA 安装
参考[开发者推荐的安装方式](https://github.com/oushujun/EDTA#installation)

**注意RepeatMasker安装的Repbase库的加入，参考RepeatMasker的安装。**

### 3.2.1. quick installation using conda
1. 下载EDTA
`git clone https://github.com/oushujun/EDTA.git`

2. 用EDTA.yml创建新的环境和安装依赖
`conda env create -f EDTA.yml`

### 3.2.2. another installation using conda
1. 【推荐】创建新的环境并进入
`conda create -n EDTA`
`conda activate EDTA`

2. 三种方式安装EDTA：
- 安装EDTA
`conda install -c bioconda -c conda-forge edta`

- 安装EDTA和指定版本依赖
`conda install -c conda-forge -c bioconda edta python=3.6 tensorflow=1.14 'h5py<3'`

- 用mamba加速安装
`conda install -c conda-forge mamba`
`mamba install -c conda-forge -c bioconda edta python=3.6 tensorflow=1.14 'h5py<3'`

### 3.2.3. singularity安装
`singularity pull EDTA.sif docker://oushujun/edta:<tag>`

### 3.2.4. docker安装
`docker pull docker://oushujun/edta:<tag>`

singularity和docker安装我没用过，参考开发者说明吧。

## 3.3. EDTA 使用
`nohup EDTA.pl --genome sample.fa --cds sample.cds --species others --step all --sensitive 1 --anno 1 --evaluate 1 -t 36 &`

**参数解释：**
- --genome 基因组文件
- --cds 提供这个种或近缘种的CDS序列（不能包括introns和UTR），用于最终过滤。
- --rmout 提供其他软件做的同源TE注释（比如repeatmasker的.out文件），如果不提供则默认使用EDTA - library用于masking。
- --species [Rice|Maize|others]三种可选
- --step	[all|filter|final|anno]
- -sensitive: 是否用RepeatModeler分析剩下的TE，默认是0，不要。RepeatModeler会增加运行时间。
- -anno: 是否在构建TE文库后进行全基因组预测，默认是0.
- -evalues: 默认是0，需要同时设置-anno 1才能使用。能够评估注释质量，但会显著增加分析时间。
- --overwrite默认是0，设定为1会删除已有结果重新运行，建议保持默认，运行中断可以继续运行。

简化推荐版：
`nohup EDTA.pl --genome sample.fa --cds sample.cds --species others --step all --anno 1 -t 36 &`


## 3.4. 结果文件
- genome.mod.EDTA.TElib.fa：最终结果，非冗余的TE库。如果在输入文件中用--curatedlib指定的修正版TE库，则该文件中也将包含这部分序列。
- genome.mod.EDTA.TElib.novel.fa: 新TE类型。该文件包括在输入文件中用--curatedlib指定的修正版TE库没有的TE序列，即genome.mod.EDTA.TElib.fa减去--curatedlib指定库(需要--curatedlib参数)。
- genome.mod.EDTA.TEanno.gff: 全基因组TE的注释。该文件包括结构完整和结构不完整的TE的注释（需要--anno 1参数）。
- genome.mod.EDTA.TEanno.sum: 对全基因组TE注释的总结（需要--anno 1参数）。
- genome.mod.MAKER.masked: 低阈值TE的屏蔽。该文件中仅包括长TE（>= 1 kb）序列(需要--anno 1参数)。
- genome.mod.EDTA.TE.fa.stat.redun.sum: 简单TE的注释偏差(需要--evaluate 1参数)。
- genome.mod.EDTA.TE.fa.stat.nested.sum：嵌套型TE注释的偏差（需要--evaluate 1参数）。
- genome.mod.EDTA.TE.fa.stat.all.sum: 注释偏差的概述（需要--evaluate 1参数）。

## 3.5. 评估已有TE注释
当已有TE库结果，想要与其他软件（比如RepeatMasker）预测结果比较，可以用EDTA的脚本`lib-test.pl`进行，评估是通过与参考注释进行比较来进行的。

1. 用RepeatMasker根据已有TE库进行注释
`RepeatMasker -pa 36 -q -no_is -norna -nolow -div 40 -lib sample.TE.lib.fasta -cutoff 225 sample.fa`
2. 用lib-test.pl进行TE注释的比较和评估
```
perl lib-test.pl -genome genome.fasta -std genome.stdlib.RM.out -tst genome.testlib.RM.out -cat [options]
	-genome	[file]	FASTA format genomesequence
	-std	[file]	RepeatMasker .out file of the standard library
	-tst	[file]	RepeatMasker .out file of the test library
	-cat	[string]	Testing TE category. Use one of LTR|nonLTR|LINE|SINE|TIR|MITEHelitron|Total|Classified
	-N	[0|1]	Include Ns in total length of the genome. Defaule: 0 (not include Ns).
	-unknown	[0|1]	Include unknownannotations to the testing category. This should be used when
				the test library has no classification and you assume they all belong to the
				target category specified by -cat. 	Default: 0 (not include unknowns)
	-rand	[int]	A randum number used to identify the current run. (default:generate automatically)
	-threads|-t	[int]	Number of threads torun this program. Default: 4
```

`lib-test.pl -genome sample.fa -std genome.stdlib.RM.out -tst genome.testlib.RM.out -cat LTR`

## 3.6. notes
- 基因组的碱基全部大写，因为小写在一些软件是被认为进行了soft-masked的，在TIR预测时会报错。
- -t设置线程，当运行到调用RMBlast引擎时，默认使用四核，如果服务器是四核，则实际使用线程为设置的-t的四倍（RepeatMasker和RepeatModeler的-pa设置线程有一样的问题），EDTA支持断点续跑，可以先设置-t 高一点，运行到RMBlast时，kill掉，重新设置低一点的-t。
- 预测有时运行会报错缺一些perl的module，例如Bio，安装缺失的module，再次运行即可。
- 关于--rmout，当使用这个参数指定同源TE注释时，不使用EDTA的同源注释，而是用这个参数指定的库做同源注释，最终结果是这个同源注释加上EDTA其他注释的完整的intact TEs结果。所以当提供的repeatmasker文件预测结果非常全面时，使用这个参数；如果提供的库对物种TE的预测结果不够全面，使用这个参数反而会降低最终预测结果，不建议使用这个参数。不知道是否提供的库是否全面，有时间就有这个参数和没这个参数的结果比较选用，没时间就直接不用这个参数。参考[issue](https://github.com/oushujun/EDTA/issues/193)。
- HelitronScanner预测Helitrons默认需要400G运行内存，若不够会出现如下报错，把运行内存上限升高后再次运行即可：

```
ERROR: Raw Helitron results not found in sample.fa.mod.EDTA.raw/sample.fa.mod.Helitron.raw.fa
	If you believe the program is working properly, this may be caused by the lack of intact Helitrons in your genome. Consider to use the --force 1 parameter to overwrite this check
```

references

1. https://en.wikipedia.org/wiki/Tandem_repeat
2. https://en.wikipedia.org/wiki/Microsatellite
3. http://www.repeatmasker.org/RepeatModeler/
4. https://www.cnblogs.com/zhanmaomao/p/12345462.html
5. https://www.jianshu.com/p/ddd1c9a74fde
6. https://www.jianshu.com/p/dfa89f394882
7. https://wheat.pw.usda.gov/ITMI/Repeats/Repeat_types.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>