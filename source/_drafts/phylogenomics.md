---
title: phylogenomics
date: 2020-09-10 09:50:00
categories: bio，phylogenomics

---  

## 建树指南
http://zhangzhen.plus/2019/11/17/simple-tutorial-of-phylogeny.html

### align

MP,NJ树不需要选择模型
BI，ML树需要做模型选择

### modelselect
不同软件对模型的支持数量差异很大，Raxml支持的模型数量很少。
iqtree可建ML树，并自动选择最优模型
partitionFinder是模型选择软件，支持分区选择，支持针对Mrbayes和Raxml进行模型选择。

#### PartitionFinder2

- install PartitionFinder.py
下载解压，并安装一些python依赖就可使用，但注意要用python2.7
- apply PartitionFinder.py
PartitionFinder.py directory -p 12
把phylip格式的aligned序列，config文件放进同一个目录，PartitionFinder.py后跟目录名即可运行，-p设定线程数。
config文件如下

```config
## ALIGNMENT FILE ## #phylip格式序列文件
alignment = plastome.phy; 

## BRANCHLENGTHS: linked | unlinked ## #枝长连锁，默认值
branchlengths = linked; 

## MODELS OF EVOLUTION: all | allx | mrbayes | beast | gamma | gammai | <list> ## #测试哪些进化模型，有时间就选all，也支持mrbayes，beast，iqtree，raxml等参数，代表测试这些软件支持的模型
models = all; 

# MODEL SELECCTION: AIC | AICc | BIC # 评判模型适合与否的标准，建议aicc或bic
model_selection = aicc;

## DATA BLOCKS: see manual for how to define ##分区块测试模型
[data_blocks]
lsc = 1-86093;
irb = 86094-112700;
ssc = 112701-129712;

## SCHEMES, search: all | user | greedy | rcluster | rclusterf | kmeans ##默认贪婪方案
[schemes]
search = greedy;
```
- results
结果在best_scheme.txt 文件中，可以直接复制适合iqtree的模型，用在iqtree上。



### 建树软件
MP建树软件：paup
ML建树软件：iqtree（快），Raxml（慢，认可准确度）
BI建树软件：MrBayes

### paup建MP树
获取align过的fasta序列，并通过mega转化成nex格式，在nex格式文件后加上如下参数设置模块，在linux运行paup test.nex即可获取test.tre的树文件
参数模块如下：
begin paup;
log file=test.log;
set autoclose=yes;
set criterion=parsimony;
set root=outgroup;
set storebrlens=yes;
set increase=auto;
outgroup out_1 out_2;
bootstrap nreps=1000 search=heuristic / addseq=random nreps=10 swap=tbr hold=1;
savetrees from=1 to=1 file=test.tre format=altnex brlens=yes savebootp=NodeLabels MaxDecimals=0;
end;
参数模块包含在begin paup和end语句之间，生成的log文件命名为test.log，criterion=parsimony是指用MP法建树，外类群out_1 out_2是nex文件中外类群的序列名称，bootstrap nreps=1000是用1000次的自展重复评估clade的支持率，结果的树文件命名为test.tre

### mrbayes建BI树
fasta/phylip文件转换成nexus文件格式，并在nexus文件最后附上设置文本，或者在mb之后交互命令一行行输入设置。
```
begin mrbayes;

	charset lsc = 1-85868;
	charset irb = 85869-112618;
	charset ssc = 112619-129655;

	partition PartitionFinder = 3:lsc, irb, ssc;
	set partition=PartitionFinder;

	lset applyto=(1) nst=6 rates=invgamma;
	lset applyto=(2) nst=6 rates=invgamma;
	lset applyto=(3) nst=6 rates=invgamma;

	prset applyto=(all) ratepr=variable;
	unlink statefreq=(all) revmat=(all) shape=(all) pinvar=(all) tratio=(all);

    mcmc nchains=24 ngen=2000000 samplefreq=1000 printfreq=500 diagnfreq=5000；
    mcmc；

    sump relburnin=yes；
    sumt relburnin=yes；

end;
``` 
建议下面这部分手动交互输入
```
    mcmc nchains=24 ngen=2000000 samplefreq=1000 printfreq=500 diagnfreq=5000；
    mcmc；

    sump relburnin=yes；
    sumt relburnin=yes；
```
mb命令进入mrbayes
>execute sample.nexus;
载入nexus文件。
>mcmc nchains=24 ngen=2000000 samplefreq=1000 printfreq=500 diagnfreq=5000;
设置mcmc运算参数
>mcmc;
mcmc运行,时间很长，会输出分裂频率分支频率的平均标准偏差average standard deviation of split frequencies，当这个值低于0.01时便可停止运行，也可以等设置的全部代数运行完毕，会提示是否继续分析。（如果小于0.01便不继续分析，如果不小于0.01便继续分析，并填写增加分析的代数。
如果未到设置的代数便小于0.01，可手动ctrl+C强行中断运行程序.

>sump
合并参数结果，relburnin=yes为舍弃抽样的前25%的结果，生成文件sample.nexus.pstat
>sumt
合并系统树，relburnin=yes为舍弃抽样的前25%的抽样数，生成文件sample.nexus.<parts|tstat|vstat|trprobs|con>
其中sample.nexus.con.tre文件含有两棵合并树（consensus trees），一棵具有每一个分支的posterior probabilities的树和一棵具有平均枝长mean branch lengths的树。
强行中断程序的情况下，要在sample.nexus.run*.t文件后增加一行End;才能sumt，否则sumt会提示error。


### raxML建ML树
nohup raxml-ng --all --msa test.fasta --model TVM+G4 --data-type DNA --bs-trees 1000 --prefix outname --threads 24 --outgroup outgroupID &
raxml-ng支持checkpoint，运行中断后使用同样的命令默认会断点重跑，如果想覆盖已有结果，使用--redo参数。

raxmlHPC-PTHREADS -s test.fas -m GTRGAMMAIX -# 1000 -x 12345 -p 12345 -T 12 -o out_1,out_2 -f a -n ml
-s test.fas #输入文件，支持phylip，fasta格式
-m GTRGAMMAI #模型参数，DNA数据常用GTR系列模型，例如GTRGAMMA,GTRCAT,GTRGAMMAI，氨基酸数据常用模型PROTGAMMALGX,
-# 1000 #bootstrap自展重复参数，通常用1000，也可设置antoMR让软件决定合适参数，一定要配合-x或者-b使用打开bootstrap功能；
-x -p #随机数种子，可随意输入几个数字
-T 12 #线程数
-o out_1,out_2 #外类群序列名称，多个用英文,隔开
-f a #选择算法，常用a，为常用的快速自展分析
-n ml #结果文件后缀为.ml，必须指定

"-f a" , "-x 12345", "-# 1000"配合使用，使用快速自展算法分析，bootstrap数量1000次
"-f d" , "-b 12345", "-# 1000"配合使用，bootstrap数量1000次，未测试



结果生成五个文件：
RAxML_bestTree.ml #最佳得分的一棵ML树
RAxML_bipartitions.ml #有bootstrap分值支持率的最佳得分树，分值在node上
RAxML_bipartionsBranchLabels.ml #有bootstrap分值支持率的最佳得分树，分值在branch labels上,figtree无法识别
RAxML_bootstrap.ml #自展重复的所有bootstrapped trees，1000次重复就有1000棵树
RAxML_info.ml #运行log文件

### iqtree
速度快，自动估算最优替代模型，支持断点重新运行

用iqtree调用ModelFinder进行模型测试
iqtree -s in.fa -m MF -T AUTO #-m MF指用ModelFinder选择模型，也可使用jModeltest或ProtTest，只需可将MF改为TEST即可，-T 设置CPU线程数。
运行完成，会在序列所在目录生成多个文件，可以打开其中日志log文件，ModelFinder会根据不同标准推荐最佳模型，而ModelFinder默认推荐使用BIC标准，同时也有AIC（不推荐）和AICc标准。选择AIC或BIC最低值对应的模型用于ML或BI建树。（值得注意的是，RaxML软件可选模型很少，有可能测试出来的模型在RaxML里不支持使用）

iqtree建树
iqtree -s in.fa -m MFP -b 1000 -alrt 1000
自展检验1000次，并获取SH-aLRT检验支持度。
-s sample.fasta/phy 比对好的多条序列
-m MFP 自动估算最优替代模型并使用，如果计算资源允许，增加-mtree，会检查所有可用模型，如果输入SNP数据，在MFP后加上+ASC，即-m MFP+ASC

评估分支支持度的方法：-B/bb -b -alrt等，可选用一个或多个
-B 1000 快速bootstrap法自展检验1000次，模型冲突情况下，快速bootstrap可能高估BS值，推荐加上-bnni参数
-b 1000 常规bootstrap法，比-bb慢，不赶时间可用
-alrt 1000 SH-aLRT检验
-pre 默认输出文件以sample.fasta为前缀，-pre参数修改输出文件前缀

生成文件
sample.fasta.iqtree # 记录相对具体的进化树构建信息
sample.fasta.treefile # 记录构建成的进化树的newick文本，important
sample.fasta.log # log文档


#### SNP数据建树 —— +ASC
使用snp数据建树时，推荐在模型MFP后加上+ASC，即-m MFP+ASC。

- 值得注意的是，+ASC模式不支持invariant sites，如果输入的SNP数据里包含invariant sites(非常常见，几乎每次都遇到)，会中断运行并报错，然后会生成一个去除了invariant sites的varsites.phy结尾的文件。
- 然后再用这个varsites.phy结尾的文件重新运行即可。

### 物种树




references
1. astral blog：https://www.jianshu.com/p/5dfa934480bb?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>
