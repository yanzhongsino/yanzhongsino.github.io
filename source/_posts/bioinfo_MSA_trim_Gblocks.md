---
title: 用Gblocks对多序列比对文件进行过滤
date: 2023-09-30
categories: 
- bioinfo
- MSA
- trim
tags:
- Multiple Sequence Alignment
- MSA
- sequence alignment
- trimming
- Gblocks

description: 记录了用Gblocks从多序列比对(Multiple Sequence Alignment,MSA)文件中过滤(trimming)保守序列的方法，以便用于后续系统发育树构建。

---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=543965304&auto=1&height=32"></iframe></div>

# 1. Gblocks简介
1. Gblocks是什么
- Gblocks 是一个用 C 语言编写的计算机程序，它通过一定的规则过滤掉 DNA 或蛋白质比对序列中比对不佳的位置和分歧较大的非保守区域。
2. Gblocks原理
- Gblocks 通过挑选序列块（blocks）来筛选比对序列，序列块（blocks）必须满足一定的要求，如缺乏大段连续的非保守位置、缺乏或低密度的GAP位置以及侧翼位置的高度保守性，从而使最终选出的blocks组成的比对序列更适合用于系统发育学分析。
3. Gblocks特点
- 删除大片段非保守性或非同源性片段（6-10bp的非同源片段识别得不好），还对block的长度进行了限制。
- Gblocks 可输出网页文件来显示哪些比对区块被保留，哪些被丢弃。
- 不足：武断地规定了某个具体阈值来判断比对片段的保留或删除，对所有基因和所有片段一刀切式的处理，没有考虑不同片段的不同进化速率。

# 2. Gblocks有多个版本
2000年发布第一版Gblocks 0.73b，后更新一版0.91b，之后就没更新过了。
1. 在线版本：http://www.phylogeny.fr/one_task.cgi?task_type=gblocks
- 如果没有特殊参数，序列较短，可使用在线版本，方便快捷。
2. 本地版本包括Mac OS X，Window，Linux三个系统的版本
- 下载地址：https://www.biologiaevolutiva.org/jcastresana/Gblocks.html
- Linux系统中解压缩解包即可得到执行文件Gblocks_0.91b

```shell
wget https://www.biologiaevolutiva.org/jcastresana/Gblocks/Gblocks_Linux64_0.91b.tar.Z
uncompress Gblocks_OS_0.91b.tar.Z
tar xvf Gblocks_OS_0.91b.tar
```

# 3. Gblocks的使用
- Gblocks有两种用法，一种是交互式，一种是命令行式。这里推荐命令行式。
- Gblocks命令行式，fasta文件在前面，后面跟着参数；如果没跟参数，则自动进入交互式界面。

## 3.1. Gblocks命令行式用法
1. 常用命令：`Gblocks samples_align.fas -t=d`，序列比对文件跟在Gblocks后，后面再接参数。
2. Gblocks命令行用法的常用参数
- `-t=`：序列类型，包括p(protein, default)、d(DNA)、c(codons)
- `-b1=`：单个保守位置（conserved position）的序列的最小数量，默认是**50%的序列总数+1**。
- `-b5=`：允许的Gap位置占比，默认是n(none)，可选h(half)或a(all)。如果是多个基因串联的序列，不是所有样本都有所有基因的序列（缺失的用--表示），这个参数推荐h或a，以保留有缺失样本的这部分基因的信息，获得更长的序列。
- `-e=`：生成文件的后缀扩展名，默认是`-gb`

3. 下表里包含了所有参数

<table>
	<thead>
		<lr>
		<th>PARAMETER NAME</th>
		<th>MEANING<br>(Default)</th>
		<th>ALLOWED VALUES</th>
	</thead>
	<tbody>
		<tr>
			<td><i>(None)</i></td>
			<td>Filename<br>(No default)</td>
			<td>Alignment or pathnames file</td>
		</tr>
		<tr>
			<td>-t=</td>
			<td>Type Of Sequence<br>(<b>Protein</b>, DNA, Codons)</td>
			<td><b>p</b>, <b>d</b>, <b>c</b></td>
		</tr>
		<tr>
			<td>-b1=</td>
			<td>Minimum Number Of Sequences For A Conserved Position<br>(<b>50% of the number of 	sequences + 1</b>)</td>
			<td>Any integer bigger than half the number of sequences and smaller or equal than the 	total number of sequences</td>
		</tr>
		<tr>
			<td>-b2=</td>
			<td>Minimum Number Of Sequences For A Flank Position<br>(<b>85% of the number of 	sequences</b>)</td>
			<td>Any integer equal or bigger than Minimum Number Of Sequences For A Conserved 	Position</td>
		</tr>
		<tr>
			<td>-b3=</td>
			<td>Maximum Number Of Contiguous Nonconserved Positions<br>(<b>8</b>)</td>
			<td>Any integer</td>
		</tr>
		<tr>
			<td>-b4=</td>
			<td>Minimum Length Of A Block<br>(<b>10</b>)</td>
			<td>Any integer equal or bigger than 2</td>
		</tr>
		<tr>
			<td>-b5=</td>
			<td>Allowed Gap Positions<br>(<b>None</b>, With Half, All)</td>
			<td><b>n</b>, <b>h</b>, <b>a</b></td>
		</tr>
		<tr>
			<td>-b6=<br><i>(Only available for protein alignments; only visible in the extended 	block parameters menu) </i></td>
			<td>Use Similarity Matrices<br>(<b>Yes</b>, No)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-b0=<br><i>(This option does not appear in the menu) </i></td>
			<td>Minimum Length Of An Initial Block<br>(<b>Same as Minimum Length Of A Block</b>)</td>
			<td>Any integer equal or bigger than 2</td>
		</tr>
		<tr>
			<td>-s=</td>
			<td>Selected Blocks<br>(<b>Yes</b>, No)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-p=</td>
			<td>Results And Parameters File<br>(<b>Yes</b>, Text, Short Text, No)</td>
			<td><b>y</b>, <b>t</b>, <b>s, n</b></td>
		</tr>
		<tr>
			<td>-v=<br><i>(Only visible in the extended saving options) </i></td>
			<td>Characters Per Line In Results And Parameters File<br>(<b>60</b>)</td>
			<td>Any integer bigger than 50</td>
		</tr>
		<tr>
			<td>-n=<br><i>(Only visible in the extended saving options) </i></td>
			<td>Nonconserved Blocks<br>(Yes, <b>No</b>)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-u=<br><i>(Only visible in the extended saving options) </i></td>
			<td>Ungapped Alignment<br>(Yes, <b>No</b>)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-k=<br><i>(Only visible in the extended saving options) </i></td>
			<td>Mask File With The Selected Blocks<br>(Yes, <b>No</b>)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-d=<br><i>(Only visible in the extended saving options) </i></td>
			<td>Postscript File With The Selected Blocks<br>(Yes, <b>No</b>)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-a=<br><i>(Only available with paths files) </i></td>
			<td>Concatenated Blocks From Alignments In Batch<br>(Yes, <b>No</b>)<br></td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-c=<br><i>(Only available with paths files) </i></td>
			<td>Concatenated Input Alignments In Batch<br>(Yes, <b>No</b>)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-w=<br><i>(Only available with paths files)</i></td>
			<td>Concatenated Ungapped Alignments In Batch<br>(Yes, <b>No</b>)</td>
			<td><b>y</b>, <b>n</b></td>
		</tr>
		<tr>
			<td>-e=</td>
			<td>Generic File Extension<br>(<b>-gb</b>)</td>
			<td>Any string with 5 or less characters</td>
		</tr>
	</tbody>
</table>

## 3.2. Gblocks结果
1. samples_align.fas-gb：按照参数trim掉不符合要求的比对，保留下保守blocks的序列
2. samples_align.fas-gb.htm：网页格式文件，里面详细展示了原始比对序列中哪些被trim，哪些保留下来

# 4. references
1. Gblocks manual：https://home.cc.umanitoba.ca/~psgendb/doc/Castresana/Gblocks_documentation.html#Installation

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>