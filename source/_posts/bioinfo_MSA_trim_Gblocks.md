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

# Gblocks过滤的流程
1. 首先，评估多重排列中每个位置的保守程度，并将其分为非保守、保守或高度保守。所有大于一定值(b3设置)的连续非保守位置都会被剔除。
2. 在余下的区块中，将对侧翼进行检查并移除位置，直到区块的两个侧翼都被高度保守的位置所包围。这样，选定的区块就会被高置信度的排列位置所锚定。
3. 然后，所有间隙Gap位置（b5设置gap的定义）都会被删除。此外，与间隙位置相邻的非保守位置也会被移除，直到达到保守位置，因为与间隙相邻的区域最难对齐。最后，间隙清理后剩余的小块(b4设置)也会被移除。
4. 对输入的比对序列的一个重要要求是，在一个保守性很好的区块内（由于移帧测序错误或该特定序列的高分歧）没有完全错配的单条序列。如果其余序列有足够多的相同点，Gblocks 就会认为该序列块是保守的，并将其选中。有一些方法可以检测到这类错误排列的序列，如 ClustalX 程序中的低分段可视化，应使用这种方法来确保排列中不包含含有大量此类片段的序列。

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
1. 常用命令：`Gblocks samples_align.fas -t=d -e=-gb`，序列比对文件跟在Gblocks后，后面再接参数。
2. Gblocks命令行用法的常用参数
- `-t=`：序列类型，包括p(protein, default)、d(DNA)、c(codons)
- `-e=`：生成文件的后缀扩展名，默认是`-gb`

## Block参数解释
1. b1：设置 "保守位置的最少序列数"，即设置定义保守位置的阈值。该值必须大于序列数的一半。该参数值越大，所选位置数量越少。
2. b2：设置 "侧翼位置最少序列数"（Minimum Number Of Sequences For A Flank Position），即设置侧翼位置定义的阈值。该值必须大于或等于 "保守位置的最少序列数"。该参数值越大，所选位置数量越少。
- 如果是多个基因串联的序列，有样本缺失一个基因的情况（缺失的用--表示），这个参数推荐变小一点，以保留有缺失样本的这部分基因的信息，获得更长的序列。
3. b3：设置 "最大连续非保守位置数"。所有连续非保守位置大于该值的线段都将被拒绝。该参数值越大，所选位置数越多。
4. b4：设置间隙清理后区块的最小长度。清除间隙后，小于此值的区块将被剔除。该参数值越大，所选位置数越少。
- 注意：在程序的旧版本（0.73b）中，有两个参数可以调节最终图块的长度。当前菜单选项所对应的参数是 "间隙清理后块的最小长度"，这是最关键的参数。另一个参数是 "初始区块的最小长度"，这个版本的程序给出的数值与之前的长度相同。不过，如果您想复制与旧版本完全相同的结果，可以通过命令行为 "初始区块最小长度 "设置任意值（见下文）。您可以通过保存选项菜单中的结果和参数文件的保存简短选项查看两个最小长度参数的值 （见下文）。
5. b5：在处理间隙位置的三种不同可能性中切换：
- 无(none/n)：最终排列中不允许出现间隙位置。在区块选择过程中，所有有一个或多个间隙的位置都会被视为间隙位置，它们和相邻的非保留位置都会被消除。
- 半数(half/h)：只有 50% 或更多序列有间隙的位置才会被视为间隙位置。因此，如果在适当的区块内有少于 50%的序列存在间隙位置，则可以在最终比对中选择这些位置。
- 全部(all/a)：可以选择所有间隙位置。有间隙的位置不会与其他位置区别对待。

## 所有参数

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
1. samples_align.fas-gb：按照参数过滤掉不符合要求的比对，保留下保守blocks的比对序列结果。
2. samples_align.fas-gb.htm：
- 网页格式文件，里面详细展示了原始比对序列中哪些被trim，哪些保留下来。可以通过查看这个文件来可视化地检查结果是否按照需求保留和过滤了，建议只要过滤都看一看。
- 文件里包含了运行Gblocks的所有参数，以及过滤前后的序列长度。
- Gblocks 生成 HTML 格式，显示了使用 ClustalW 1.7 程序和默认参数从 nad3.pir 对齐结果中选择的区块。根据 Gblocks 所给参数定义的保留位置被高亮显示，用颜色显示重复次数最多的氨基酸，选定的位置用蓝色粗线下划。氨基酸的颜色（仅作为视觉指南，不代表任何距离矩阵）为：A、G、S 和 T 为石灰色；P 为水绿色；C 为橙色；D、E、Q 和 N 为白色；F、W 和 Y 为黄色；H、K 和 R 为红色；I、L、M 和 V 为紫红色。

# 4. references
1. Gblocks manual：https://home.cc.umanitoba.ca/~psgendb/doc/Castresana/Gblocks_documentation.html#Installation

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>