---
title: 从codon比对的cds序列中提取四倍简并位点
date: 2021-10-10 10:39:54
categories:
- bio
- bioinfo
tags:
- tutorial
- codon
- 4dtv

description: 从按照密码子规则（即三个三个对齐）比对的cds序列中提取四倍简并位点(fourfold degenerate codons, 4dtv)。
---

# 1. 从condon比对的cds序列中提取四倍简并位点
输入文件是根据密码子规则（即三个三个对齐）比对好的cds序列，输出文件是以其中一个物种的4dtv为标准提取所有物种的4dtv位点，提取的也是按照比对前的顺序排列，结果保存在4dtv.aln文件中。

## 1.1. 输入数据和变量定义
1. `cds=/path/to/cds.aln` # 定义变量cds为codon模式比对好的cds序列
2. `species=Athaliana` #以哪个物种的4dtv为标准提取，就定义变量species为那个物种的序列ID

## 1.2. 三种方案【选择一种】
### 1.2.1. 一步生成
1. 一步生成运行文件，生成从cds获取4dtv的命令，很长，所以储存在文件4dtv.sh中。运行4dtv.sh就可以获取
`seqkit locate -V 0 -i -d -p GCN -p CGN -p GGN -p CTN -p CCN -p TCN -p ACN -p GTN ${cds} |awk -v awka="${species}" '{if ($1 == awka && $6%3== 0) print $6}'|sort -k 1n |uniq |awk -v awkb="${cds}" '{print "<(seqkit subseq -r "$1":"$1" "awkb")"}' |sed -e '1i\seqkit concat -w 0' -e '$a\> 4dtv.aln'|xargs echo >4dtv.sh`

代码含义：用seqkit locate根据4dtv的规则提取位置，然后筛选指定物种的位置，筛选第三位位置是3的整数倍的位置，把第三位位置(即4dtv位点)提取出来，排序去重；再按照seqkit concat要求的格式规则把seqkit concat命令写入提取脚本4dtv.sh中。

除非位点在1000个以内，否则一步生成运行文件可能会运行失败，推荐用下一种方案。

2. 运行生成的4dtv.sh，获得4dtv.aln结果
`bash 4dtv.sh`

### 1.2.2. 切割分开运行【推荐】
如果需要合并的位点超过1000个，一步生成运行文件占用内存，运行较慢，可以分步骤进行。

如果因为4dtv.sh太长无法运行，可以把4dtv.temp分成1-n几部分（比如1000行作为一个子文件），然后切割各部分分别生成4dtv_n.sh，之后分别运行再合并。
1. 生成临时文件4dtv.temp
`seqkit locate -V 0 -i -d -p GCN -p CGN -p GGN -p CTN -p CCN -p TCN -p ACN -p GTN ${cds} |awk -v awka="${species}" '{if ($1 == awka && $6%3== 0) print $6}' |sort -k 1n |uniq >4dtv.temp`

2. 分割临时文件4dtv.temp
`split -l 1000 4dtv.temp -d -a 4 4dtv.temp_`
命令含义：分割文件4dtv.temp成多个文件，每个子文件1000行（-l 1000），以数字（-d）而非字母命名子文件，数字长度为4（-a 4，default是2）；会生成4dtv.temp_0000-nnnn的子文件。

可以先看看4dtv.temp的长度，我尝试了下按1000行切割比较合适，再多就达到能同时打开文件数量上限无法运行了。

3. 生成4dtv.temp.n.sh
`for i in $(ls 4dtv.temp_*);do cat ${i} |awk -v awkb="${cds}" '{print "<(seqkit subseq -r "$1":"$1" "awkb")"}' |sed -e '1i\seqkit concat -w 0' -e '$a\>'|sed "s/^>$/> ${i}.aln/" |sed 's/$/ \\/g' |sed '$ s/.$//' >${i}.sh; done`

4. 运行生成的4dtv.temp.n.sh，获得4dtv.aln_n结果【耗时步骤】
`for i in $(ls 4dtv.temp_*.sh);do bash ${i};done`

建议用nohup放到后台运行
`nohup sh -c 'for i in $(ls 4dtv.temp_*.sh);do bash ${i};done' &`

5. 合并子文件，获得最终结果
`seqkit concat ./4dtv.temp_*.aln >4dtv.aln`

可以`seqkit stats *aln`检查是否都是1000长度的序列。

6. 删除中间文件
`rm 4dtv.temp*`

### 1.2.3. 分物种运行【不推荐】
一开始没发现seqkit concat这个工具，然后自己写的合并，有时会有aln的问题，不建议用。
1. 定义数组sps为物种列表
`sps=($(seqkit seq -n ${cds}|xargs))`
2. 以第六个物种${sps[5]}的4dtv为标准，为每个物种保存一份4dtv.bed文件
`for i in $(echo ${sps[*]}); do cat ${sps[5]}.4dtv|sed "s/${species[5]}/${i}/g" >${i}.4dtv.bed; done`
3. 为每个物种提取4dtv位点，保存到4dtv.aln.temp【耗时步骤】
`for i in $(echo ${sps[*]}); do seqkit subseq --bed ${i}.4dtv.bed ${cds} | grep -v ">"|xargs echo |sed "1i\>${i}" >${i}.4dtv.aln.temp; done`
4. 合并物种的4dtv位点
`cat *.4dtv.aln.temp >4dtv.aln` #合并
