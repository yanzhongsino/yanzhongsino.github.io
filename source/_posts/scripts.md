---
title: scripts
date: 2021-04-01 10:56:00
categories: 
- computer
- scripts
tags: 
- scripts
- biosoft
- batch_run.sh
- ROUSFinder2.0.py
- extract_intron_info.pl
- multi-transcriptome_assembly_with_ref.sh
- extract_single_copy_genes.sh

description: store some scripts using in biology and bioinformatics
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=476751845&auto=1&height=32"></iframe></div>

# 1. 我写的脚本
## 1.1. 多个RNA-seq数据，基于参考基因组的转录本组装和ORF预测
[multi-transcriptome_assembly_with_ref.sh](https://github.com/yanzhongsino/bioscripts/blob/main/my_scripts/multi-transcriptome_assembly_with_ref.sh)

## 1.2. 提取，合并，和比对单拷贝基因和蛋白
[extract_single_copy_genes.sh](https://github.com/yanzhongsino/bioscripts/blob/main/my_scripts/extract_single_copy_genes.sh)

# 2. 我修改的脚本（其他人写的）
## 2.1. 从包含mRNA和CDS的gff注释文件中提取intron的位置和长度信息
[extract_intron_info.pl](https://github.com/yanzhongsino/bioscripts/blob/main/modifiedscripts/extract_intron_info.pl)

# 3. 保存的脚本
## 3.1. batch_run.sh —— 多线程并行运行批量化命令
[batch_run.sh](https://github.com/yanzhongsino/bioscripts/blob/main/saved_scripts/batch_run.sh)

```shell
#!/bin/bash
# batch_run.sh脚本用于多线程并行批量化循环命令，并控制并行数量始终为${thread}。
# usage: 修改thread=后的值为想要设定的并行线程数，把需要并行的命令替换23行的run something。运行脚本batch_run.sh，实现并行运行循环任务。
# ref：网上找的脚本，来源找不到了，侵删。

thread=16

start_time=`date +%s`              # 定义脚本运行的开始时间
echo ${start_time}
tmpFifo=/tmp/$$.fifo # 声明管道名称，$$表示脚本当前运行的进程PID
mkfifo ${tmpFifo} # 创建有名管道
exec 3<>${tmpFifo}                   #创建文件描述符，以可读（<）可写（>）的方式关联管道文件，这时候文件描述符3就有了有名管道文件的所有特性
rm -rf ${tmpFifo}                    #关联后的文件描述符拥有管道文件的所有特性,所以这时候管道文件可以删除，我们留下文件描述符来用就可以了
for ((i=1;i<=${thread};i++))
do
        echo "" >&3                   #&3代表引用文件描述符3，这条命令代表往管道里面放入了一个"令牌"
done

for sample in $(cat sample_list.txt) #并行运行的循环
do
        read -u3                           #代表从管道中读取一个令牌
        {
                echo ${sample}
                run something # 需要并行运行的真实命令
                echo 'success' ${sample}       
                echo "" >&3                   #代表我这一次命令执行到最后，把令牌放回管道
        } &
done

## ps:可以使用while read line; do echo $line; done < file.txt代替for i in $(cat file.txt);do echo $i; done循环。但是不能用cat file.txt | while read line; do echo $line; done代替。

wait
 
stop_time=`date +%s`  #定义脚本运行的结束时间
 
echo "TIME:`expr $stop_time - $start_time`"
exec 3<&-                       #关闭文件描述符的读
exec 3>&-                       #关闭文件描述符的写
```

## 3.2. ROUSFinder2.0.py —— 植物线粒体基因组重复序列注释脚本
1. 背景
文章[Repeats of Unusual Size in plant mitochondrial genomes: identification, incidence and evolution](https://academic.oup.com/g3journal/article/9/2/549/6026745)总结了植物线粒体基因组的重复序列的特征。研究表明，植物线粒体基因组比动物的要大，包含大量的非编码DNA，突变率低，重排率高。

文章里提供了一个python脚本[ROUSFinder2.0.py](https://gsajournals.figshare.com/articles/figure/Supplemental_Material_for_Wynn_and_Christensen_2018/7425680?file=13750433)用于注释植物线粒体基因组中的非串联重复序列(non-tandem repeat)。

2. 脚本
脚本是python2写的，依赖主要是blastn。

```python
# ROUSFinder2.0.py用于植物线粒体基因组重复序列的注释
#! /usr/bin/env python  
import sys, math, os, argparse, csv  
csv.field_size_limit(sys.maxsize)  
  
# Version 2.0, November 21, 2018  
# Changes: uses variable parameters  
# Find dispersed repeated sequences in genomes.   
# Designed for plant mitochondrial genomes of up to a few Mbp.  
# May be very slow with larger genomes.   
# Blast can also sometimes give odd results with large or highly repetetive genomes.  
# Gaps, or runs of 'N's in the sequence will definitely give weird results.   
# The program assumes there aren't any, and that the longest repeat will be the full sequence to itself.  
# If there are long repeats in the output that are listed as being only at one location, this is probably what happened.  
# If there are a lot of repeats within repeats the results can also be odd.  
# Copyright Alan C. Christensen, University of Nebraska, 2018  
# No guarantees, warranties, support, or anything else is implicit or explicit.  
# Input is a fasta format file of a sequence. Genbank format works but generates lots of error messages in stdout.  
# Output is a list of unique, ungapped repeated sequences, fasta formatted.  
# The names are in the format '>Repeat/ROUS_name_start_end_length'.  
# A table of repeats with the coordinates of each one is generated.  
# A list of repeat name, length and copy number is generated.  
# A binned table of the total number of repeats in size ranges is generated.  
#  
# PARAMETERS  
#   REQUIRED:  
#      input file in fasta format  
#   Optional  
#      -o output file name  
#      -m minimum length of exact matches to keep  
#      -b path to blastn (default is /usr/bin/)  
#      -k keep temp files  
#      -gb to write the repeats to a genbank format file  
#      -rew reward for match (default is 1)  
#      -pen penalty for mismatch (default is 20)  
  
parser = argparse.ArgumentParser(description='Find repeats in a fasta sequence file')  
parser.add_argument('infile', action='store', help='Input .fasta file')  
parser.add_argument('-o', action='store', dest='outfile', help='Output file name seed, default is input_repeats', default='default')  
parser.add_argument('-m', action='store', dest='minlen', help='Minimum length of matches to keep, default=50', default='50')  
parser.add_argument('-b', action='store', dest='blast_path', help='Path to blastn program, default is /usr/bin/', default='/usr/bin/')  
parser.add_argument('-k', action='store_true', dest='keep', help='True to keep temp files', default=False)  
parser.add_argument('-gb', action='store_true', dest='genbank', help='True to write GenBank format file', default=False)  
parser.add_argument('-rew', action='store', dest='reward', help='Reward for match', default='1')  
parser.add_argument('-pen', action='store', dest='penalty', help='Penalty for mismatch', default='20')  
results = parser.parse_args()  
infile = results.infile  
outfile = results.outfile  
minlen = int(results.minlen)  
blast_path = results.blast_path  
keep = results.keep  
genbank = results.genbank  
reward = results.reward  
penalty = results.penalty  
  
# It might be useful to define the wordsize as something less than minlen, so both variables are used.  
# Wordsize smaller than minlen would give smaller core identical sequences in the middle of repeats.  
# An example might be to change this to wordsize = str(int(minlen/2)).  
wordsize = str(minlen)  
  
# If no output file seed is specified, make one by stripping leading directory information  
# and stripping trailing .fa or .fasta from the input file name and using that.  
if outfile == 'default':  
    outfile = infile  
    if outfile.count('/') > 0:  
        for i in range(outfile.count('/')):  
            index = outfile.index('/')  
            outfile = outfile[index+1:]  
    if outfile.endswith('.fa') or outfile.endswith('.fasta'):  
        outfile = outfile.rstrip('fasta')  
    outfile = outfile.rstrip('.')  
outfa = outfile+'_rep.fasta'  
outtab = outfile+'_rep_table.txt'  
outbin = outfile+'_binned.txt'  
outcount = outfile+'_rep_counts.txt'  
outgb = outfile+'_repeats.gb.txt'  
tempblast = outfile+'_tempblast.txt'  
temprepeats = outfile+'_temprepeats.txt'  
tempparse = outfile+'_sequence_parsing.txt'  
  
# Get sequence name and length from fasta file.  
seq = open(infile, 'r')  
seqname = seq.readline()  
seqname = seqname.lstrip('> ')  
seqname = seqname.rstrip()  
seqlen = 0   
for line in seq:  
    if(line[0] == ">"):  
        continue  
    seqlen += len(line.strip())  
seq.close()  
  
# run blastn with query file plus strand (removing first line which is full length sequence), minus strand, and concatenate  
print 'Performing self-blastn comparison with '+seqname      
os.system(blast_path+'blastn -query '+infile+' -strand plus -subject '+infile+' -word_size '+wordsize+' -reward '+reward+' -penalty -'+penalty+' -ungapped -dust no -soft_masking false -evalue 10  -outfmt "10 qstart qend length sstart send mismatch sstrand qseq" | tail -n+2 > tempblast1.txt')  
os.system(blast_path+'blastn -query '+infile+' -strand minus -subject '+infile+' -word_size '+wordsize+' -reward '+reward+' -penalty -'+penalty+' -ungapped -dust no -soft_masking false -evalue 10 -outfmt "10 qstart qend length sstart send mismatch sstrand qseq" > tempblast2.txt')  
os.system('cat tempblast1.txt tempblast2.txt > '+tempblast)  
os.system('rm tempblast1.txt tempblast2.txt')  
  
# open tempblast.txt, convert to list of lists, and sort by length and position descending  
# This is necessary because blastn does not output every possible pair of hits when there are more than 2 copies of a repeat  
  
print 'Sorting alignments...'  
f = open(tempblast, 'r')  
reader = csv.reader(f)  
alignments = list(reader)  
f.close()  
alignments = sorted(alignments, key=lambda x: (-1*int(x[2]), -1*int(x[0])))  
alignments.append(['1','1','1','1','1','0','A','X'])  
  
# New list of uniques  
# Text file '_sequence_parsing.txt' includes the information on how duplicates were found.  
# Start at row 0. Compare to subsequent rows.   
# If repeat length is different from the next row, it has passed all the tests, write it to the file.  
# If query or subject coordinates are the same as the query or subject or reversed coordinates  
# of a subsequent row, it is not unique, so go to the next row and do the comparisons again.  
# Thanks to Alex Kozik for repeatedly testing and finding bugs in the algorithm.  
print 'Finding unique repeats...'  
uniques = []  
sp = open(tempparse, 'w')  
for row in range(len(alignments)):  
    sp.write('row '+str(row)+'\n')  
      
    if int(alignments[row][2]) < minlen:  
        # This won't happen unless the word_size is defined as something other than minlen.  
        # That could be useful under some circumstances.  
        sp.write('row '+str(row)+' is less than minlength')  
        break  
    else:  
      
        for compare in range(row+1,len(alignments)):  
            if alignments[row][2] != alignments[compare][2]:   
                uniques.append(alignments[row])  
                sp.write('\tadding row '+str(row)+' to unique list\n')  
                break  
            else:  
                sp.write('\tcomparing to '+str(compare)+'\n')  
      
                if alignments[row][0] == alignments[compare][0] and alignments[row][1] == alignments[compare][1]:  
                    sp.write('\tqstart and qend of row '+str(row)+' and '+str(compare)+' are the same\n')  
                    break  
                elif alignments[row][0] == alignments[compare][1] and alignments[row][1] == alignments[compare][0]:  
                    sp.write('\tqstart and qend of row '+str(row)+' is the same as qend and qstart of '+str(compare)+'\n')  
                    break  
                elif alignments[row][0] == alignments[compare][3] and alignments[row][1] == alignments[compare][4]:  
                    sp.write('\tqstart and qend of row '+str(row)+' is the same as sstart and send of '+str(compare)+'\n')  
                    break  
                elif alignments[row][0] == alignments[compare][4] and alignments[row][1] == alignments[compare][3]:  
                    sp.write('\tqstart and qend of row '+str(row)+' is the same as send and sstart of '+str(compare)+'\n')  
                    break  
                elif alignments[row][3] == alignments[compare][0] and alignments[row][4] == alignments[compare][1]:  
                    sp.write('\tsstart and send of row '+str(row)+' is the same as qstart and qend of '+str(compare)+'\n')  
                    break  
                elif alignments[row][3] == alignments[compare][1] and alignments[row][4] == alignments[compare][0]:  
                    sp.write('\tsstart and send of row '+str(row)+' is the same as qend and qstart of '+str(compare)+'\n')  
                    break  
                elif alignments[row][3] == alignments[compare][3] and alignments[row][4] == alignments[compare][4]:  
                    sp.write('\tsstart and send of row '+str(row)+' is the same as sstart and send of '+str(compare)+'\n')  
                    break  
                elif alignments[row][3] == alignments[compare][4] and alignments[row][4] == alignments[compare][3]:  
                    sp.write('\tsstart and send of row '+str(row)+' is the same as send and sstart of '+str(compare)+'\n')  
                    break  
                else:  
                    sp.write('\t'+str(row)+' is different\n')  
  
sp.close()  
  
# Write uniques into output file  
# Start list for copy number table  
rous_count = 0  
g = open(outfa, 'w')  
repcopies = []  
  
for i in range(len(uniques)):  
    qstart = uniques[i][0]  
    qend = uniques[i][1]  
    length = uniques[i][2]  
    seq = uniques[i][7]  
      
    rous_count += 1  
    g.write('>Repeat_'+str(rous_count)+'\n'+seq+'\n')  
    repcopies.append(['Repeat_'+str(rous_count),length])  
          
if rous_count == 0:  
    print "\tRepeats of unusual size? I don't think they exist"  
g.close()  
print 'Repeat fasta file is done, as you wish.'  
  
# Now find each copy of each repeat. Again, this is because the blastn output file does not have every possible alignment.  
# It is also because the information on locations and strand is not organized well in the blastn output.  
# In addition, this subroutine eliminates duplicates of nested repeats.  
  
print "Finding all copies of repeats..."  
g = open(outfa, 'r')  
os.system(blast_path+'blastn -query '+outfa+' -strand both -subject '+infile+' -word_size '+wordsize+' -reward 1 -penalty -20 -ungapped -dust no -soft_masking false -evalue 1000 -outfmt "10 qseqid length sstart send sstrand qcovhsp" > '+temprepeats)  
g.close()  
  
tempr = open(temprepeats, 'r')  
reader = csv.reader(tempr)  
replist = list(reader)  
tempr.close()  
  
print "Making a table of the repeats..."  
sum_rep_len = 0  
bin_dict = {}  
binned = [seqname,seqlen,0]  
  
# defining the bins  
i = 0  
j = 50  
while j < 1000:  
    bin_dict[i] = j  
    binned.append(0)  
    i += 1  
    j += 50  
while j <= 10000:  
    bin_dict[i] = j  
    binned.append(0)  
    i +=1  
    j += 250  
      
# make list for entire sequence, set each position as 0  
posit = []  
for n in range(seqlen):  
    posit.append(0)  
  
# Thanks to Emily Wynn for suggesting qcovhsp for this loop.  
# if qcovhsp is >98%, write to the file  
# write tab separated values of repeat name, length, start, end, strand to outtab  
# make list for genbank file  
# Keep stats on lengths  
rt = open(outtab, 'w')  
rt.write(seqname+'\t'+str(seqlen)+'\n')  
templist = []  
gblist =[]  
  
# look at each repeat in turn  
for i in range(len(replist)):  
    # if repeat is good (>98% identical to another one), write it to the file, and put the name in a list  
    if int(replist[i][5])>98:  
        rt.write(str(replist[i][0])+'\t'+str(replist[i][1])+'\t'+str(replist[i][2])+'\t'+str(replist[i][3])+'\t'+str(replist[i][4])+'\n')  
        if replist[i][4] == 'minus':  
            location = 'complement('+replist[i][3]+'..'+replist[i][2]+')'  
        else:  
            location = replist[i][2]+'..'+replist[i][3]  
        gblist.append('     repeat_region   '+location+'\n                     /rpt_type=dispersed\n                     /label='+replist[i][0]+'\n')  
        templist.append(replist[i][0])  
        # then write 1's at every position in the sequence covered by that repeat  
        # these can then be summed to get total bases of repeats  
        # bases in overlapping repeats are only counted once  
        for n in range(int(replist[i][2]), int(replist[i][3])):  
            posit[n] = 1  
        # then scan through bin sizes and if a repeat is greater than the  
        # bin_dict size cutoff, add one to the bin  
        for j in range(len(binned)-4, -1, -1):  
            if int(replist[i][1]) >= bin_dict[j]:  
                binned[j+3] +=1  
                break  
sum_rep_len = posit.count(1)  
binned[2] = sum_rep_len  
rt.close()  
if genbank == True:  
    gb = open(outgb, 'w')  
    for i in range(len(gblist)):  
        gb.write(gblist[i])  
    gb.close()  
  
# write tab separated values of repeat name, length, copy number to outcount  
# first two lines are also a table of stats on repeats  
rc = open(outcount,'w')  
rc.write('Sequence\tGenome_size\tNumROUS\tAvgSize\tAvgCopyNum\n')  
  
numrous = 0  
sizerous = 0  
copyrous = 0  
  
for i in range(len(repcopies)):  
    repname = repcopies[i][0]  
    replen = float(repcopies[i][1])  
    repcop = float(templist.count(repname))  
  
    numrous += 1  
    sizerous += replen  
    copyrous += repcop  
  
if numrous == 0:  
    avsizerous = 'NA'  
    avcopyrous = 'NA'  
else:  
    avsizerous = sizerous/numrous  
    avcopyrous = copyrous/numrous  
  
  
rc.write(seqname+'\t'+str(seqlen)+'\t'+str(numrous)+'\t'+str(avsizerous)+'\t'+str(avcopyrous)+'\n')  
  
for i in range(len(repcopies)):  
    rc.write(repcopies[i][0]+'\t'+repcopies[i][1]+'\t'+str(templist.count(repcopies[i][0]))+'\n')  
  
rc.close()  
  
# Write binned table headers, then stats for this sequence.  
binfile = open(outbin, 'w')  
binfile.write('Sequence\tSeq_len\tRep_len\t')  
for i in range(len(bin_dict)):  
    binfile.write(str(bin_dict[i])+'\t')  
binfile.write('\n')  
for i in range(len(binned)):  
    binfile.write(str(binned[i])+'\t')  
binfile.write('\n')  
binfile.close()  
print "Repeat tables are done, as you wish."  
  
# Removing temp files if necessary  
if keep == False:  
    os.system('rm '+tempblast+' '+temprepeats+' '+tempparse)  
  
# Rachael Schulte, William Goldman and Rob Reiner inspired this section of code  
quote_dict = {0:"48656c6c6f2e204d79206e616d6520697320496e69676f204d6f6e746f79612e20596f75206b696c6c6564206d79206661746865722e205072657061726520746f206469652e", 1:"5768656e20492077617320796f7572206167652c2074656c65766973696f6e207761732063616c6c656420626f6f6b732e", 2:"486176652066756e2073746f726d696e2720646120636173746c6521", 3:"4d79207761792773206e6f7420766572792073706f7274736d616e6c696b652e", 4:"596f75206b656570207573696e67207468617420776f72642e204920646f206e6f74207468696e6b206974206d65616e73207768617420796f75207468696e6b206974206d65616e732e", 5:"4d75726465726564206279207069726174657320697320676f6f642e",6:"496e636f6e6365697661626c6521", 7:"5468657265277320612062696720646966666572656e6365206265747765656e206d6f73746c79206465616420616e6420616c6c20646561642e", 8:"596f7520727573682061206d697261636c65206d616e2c20796f752067657420726f7474656e206d697261636c65732e", 9:"476f6f64206e696768742c20576573746c65792e20476f6f6420776f726b2e20536c6565702077656c6c2e2049276c6c206d6f7374206c696b656c79206b696c6c20796f7520696e20746865206d6f726e696e672e",10:"4e6f206d6f7265207268796d65732c2049206d65616e2069742120416e79626f64792077616e742061207065616e75743f"}  
import random, binascii  
z = random.randint(0,10)  
print binascii.unhexlify(quote_dict[z])+'\n'  
```

3. 运行
`python2 ROUSFinder2.0.py mito.genome.fa -m 100 -b /path/to/blastn/ -gb -o sample`

- -m 100：指定注释重复序列的最小长度为100bp，默认是50bp。
- -b /path/to/blastn/：参数指定blastn的所在路径，默认是/usr/bin/。
- -o sample：指定输出文件前缀。
- -gb：生成GenBank格式文件。

4. 输出文件
输出文件有四个，mito.genome.fa_binned.txt, mito.genome.fa_rep.fasta, mito.genome.fa_rep_counts.txt, mito.genome.fa_rep_table.txt。

如果使用了-gb还会生成mito.genome.fa_repeats.gb.txt。

- mito.genome.fa_rep_table.txt文件，内容包括重复序列的长度，起始位置，正负链等信息。
- mito.genome.fa_rep.fasta是鉴定出来的重复序列。
- mito.genome.fa_rep_counts.txt是重复序列的数量信息，包括序列ID-Sequence,基因组尺寸-Genome_size,重复序列数量-NumROUS,重复序列平均尺寸-AvgSize,平均重复次数-AvgCopyNum，以及每个重复序列的长度信息。
- mito.genome.fa_binned.txt，其他统计信息。
- mito.genome.fa_repeats.gb.txt是GeneBank格式的重复序列文件。

## infernal-tblout2gff.pl —— 基因组ncRNA的infernal注释结果tblout格式转换成gff格式
1. 背景
用Infernal注释基因组的ncRNA，得到的结果可以用脚本整理成gff3格式。

脚本来源：https://www.cnblogs.com/jessepeng/p/15392809.html

2. 脚本
perl脚本
```perl
#!/usr/bin/env perl
# infernal-tblout2gff.pl: convert cmsearch or cmscan tblout files to GFF format

# an important point of above ref:
# "Start is always less than or equal to end"
# EPN, Fri Jun  7 11:07:38 2019
# 
#
use strict;
use warnings;
use Getopt::Long;

my $in_tblout  = "";   # name of input tblout file

my $usage;
$usage  = "infernal-tblout2gff.pl\n\n";
$usage .= "Usage:\n\n";
$usage .= "infernal-tblout2gff.pl [OPTIONS] <cmsearch tblout file>\n\tOR\n";
$usage .= "infernal-tblout2gff.pl --cmscan [OPTIONS] <cmscan tblout file>\n\tOR\n";
$usage .= "infernal-tblout2gff.pl --cmscan --fmt2 [OPTIONS] <cmscan --fmt 2 tblout file>\n\n";
$usage .= "\tOPTIONS:\n";
$usage .= "\t\t-T <n>       : minimum bit score to include is <n>\n";
$usage .= "\t\t-E <x>       : maximum E-value to include is <x>\n";
$usage .= "\t\t--cmscan     : tblout file was created by cmscan\n";
$usage .= "\t\t--source <s> : specify 'source' field should be <s> (e.g. tRNAscan-SE)\n";
$usage .= "\t\t--fmt2       : tblout file was created with cmscan --fmt 2 option\n";
$usage .= "\t\t--all        : output all info in 'attributes' column   [default: E-value]\n";
$usage .= "\t\t--none       : output no info in 'attributes' column    [default: E-value]\n";
$usage .= "\t\t--desc       : output desc field in 'attributes' column [default: E-value]\n";
$usage .= "\t\t--version <s>: append \"-<s>\" to 'source' column\n";
$usage .= "\t\t--extra <s>  : append \"<s>;\" to 'attributes' column\n";
$usage .= "\t\t--hidedesc   : do not includ \"desc\:\" prior to desc value in 'attributes' column\n";

my $do_minscore = 0;       # set to '1' if -T used
my $do_maxevalue = 0;      # set to '1' if -E used
my $minscore   = undef;    # defined if if -T used
my $maxevalue  = undef;    # defined if -E used
my $do_cmscan  = 0;        # set to '1' if --cmscan used
my $do_fmt2    = 0;        # set to '1' if --fmt
my $do_all_attributes = 0; # set to '1' if --all
my $do_no_attributes  = 0; # set to '1' if --none
my $do_de_attributes  = 0; # set to '1' if --desc
my $version = undef;       # defined if --version used
my $extra = undef;         # defined if --extra used
my $do_hidedesc = 0;       # set to 1 if --hidedesc used
my $opt_source = undef;    # set to <s> if --source <s> used

&GetOptions( "T=s"       => \$minscore,
             "E=s"       => \$maxevalue,
             "cmscan"    => \$do_cmscan,
             "source=s"  => \$opt_source,
             "fmt2"      => \$do_fmt2,
             "all"       => \$do_all_attributes,
             "none"      => \$do_no_attributes,
             "desc"      => \$do_de_attributes,
             "version=s" => \$version, 
             "extra=s"   => \$extra,
             "hidedesc"  => \$do_hidedesc);

if(scalar(@ARGV) != 1) { die $usage; }
my ($tblout_file) = @ARGV;

if(defined $minscore)  { $do_minscore  = 1; }
if(defined $maxevalue) { $do_maxevalue = 1; }
if($do_minscore && $do_maxevalue) { 
  die "ERROR, -T and -E cannot be used in combination. Pick one.";
}
if(($do_all_attributes) && ($do_no_attributes)) { 
  die "ERROR, --all and --none cannot be used in combination. Pick one.";
}
if(($do_all_attributes) && ($do_de_attributes)) { 
  die "ERROR, --all and --desc cannot be used in combination. Pick one.";
}
if(($do_no_attributes) && ($do_de_attributes)) { 
  die "ERROR, --none and --desc cannot be used in combination. Pick one.";
}
if(($do_fmt2) && (! $do_cmscan)) { 
  die "ERROR, --fmt2 only makes sense in combination with --cmscan"; 
}
if((defined $opt_source) && (defined $version)) { 
  die "ERROR, --source and --version are incompatible";
}

if(! -e $tblout_file) { die "ERROR tblout file $tblout_file does not exist"; }
if(! -s $tblout_file) { die "ERROR tblout file $tblout_file is empty"; }

my $source = ($do_cmscan) ? "cmscan" : "cmsearch";
if(defined $version)    { $source .= "-" . $version; }
if(defined $opt_source) { $source = $opt_source; }

open(IN, $tblout_file) || die "ERROR unable to open $tblout_file for reading"; 
my $line;
my $i;
while($line = <IN>) { 
  if($line !~ m/^\#/) { 
    chomp $line;
    my @el_A = split(/\s+/, $line);
    my $nfields = scalar(@el_A);
    if((! $do_fmt2) && ($nfields < 18)) { 
      die "ERROR expected at least 18 space delimited fields in tblout line (fmt 1, default) but got $nfields on line:\n$line\n"; 
    }
    if(($do_fmt2) && ($nfields < 27)) { 
      die "ERROR expected at least 27 space delimited fields in tblout line (fmt 2, --fmt2) but got $nfields on line:\n$line\n"; 
    }
    # ref Infernal 1.1.2 user guide, pages 59-61
    my $idx     = undef;
    my $seqname = undef;
    my $seqaccn = undef;
    my $mdlname = undef;
    my $mdlaccn = undef;
    my $clan    = undef;
    my $mdl     = undef;
    my $mdlfrom = undef;
    my $mdlto   = undef;
    my $seqfrom = undef;
    my $seqto   = undef;
    my $strand  = undef;
    my $trunc   = undef;
    my $pass    = undef;
    my $gc      = undef;
    my $bias    = undef;
    my $score   = undef;
    my $evalue  = undef;
    my $inc     = undef;
    my $olp     = undef;
    my $anyidx  = undef;
    my $anyfrct1= undef;
    my $anyfrct2= undef;
    my $winidx  = undef;
    my $winfrct1= undef;
    my $winfrct2= undef;
    my $desc    = undef;

    if($do_fmt2) { # 27 fields
      $idx     = $el_A[0];
      $seqname = ($do_cmscan) ? $el_A[3] : $el_A[1];
      $seqaccn = ($do_cmscan) ? $el_A[4] : $el_A[2];
      $mdlname = ($do_cmscan) ? $el_A[1] : $el_A[3];
      $mdlaccn = ($do_cmscan) ? $el_A[2] : $el_A[4];
      $clan    = $el_A[5];
      $mdl     = $el_A[6];
      $mdlfrom = $el_A[7];
      $mdlto   = $el_A[8];
      $seqfrom = $el_A[9];
      $seqto   = $el_A[10];
      $strand  = $el_A[11];
      $trunc   = $el_A[12];
      $pass    = $el_A[13];
      $gc      = $el_A[14];
      $bias    = $el_A[15];
      $score   = $el_A[16];
      $evalue  = $el_A[17];
      $inc     = $el_A[18];
      $olp     = $el_A[19];
      $anyidx  = $el_A[20];
      $anyfrct1= $el_A[21];
      $anyfrct2= $el_A[22];
      $winidx  = $el_A[23];
      $winfrct1= $el_A[24];
      $winfrct2= $el_A[25];
      $desc    = $el_A[26]; 
      for($i = 27; $i < $nfields; $i++) { $desc .= "_" . $el_A[$i]; }
    }
    else { # fmt 1, default
      $seqname = ($do_cmscan) ? $el_A[2] : $el_A[0];
      $seqaccn = ($do_cmscan) ? $el_A[3] : $el_A[1];
      $mdlname = ($do_cmscan) ? $el_A[0] : $el_A[2];
      $mdlaccn = ($do_cmscan) ? $el_A[1] : $el_A[3];
      $mdl     = $el_A[4];
      $mdlfrom = $el_A[5];
      $mdlto   = $el_A[6];
      $seqfrom = $el_A[7];
      $seqto   = $el_A[8];
      $strand  = $el_A[9];
      $trunc   = $el_A[10];
      $pass    = $el_A[11];
      $gc      = $el_A[12];
      $bias    = $el_A[13];
      $score   = $el_A[14];
      $evalue  = $el_A[15];
      $inc     = $el_A[16];
      $desc    = $el_A[17]; 
      for($i = 18; $i < $nfields; $i++) { $desc .= "_" . $el_A[$i]; }
    }
    # one sanity check, strand should make sense
    if(($strand ne "+") && ($strand ne "-")) { 
      if(($do_fmt2) && (($seqfrom eq "+") || ($seqfrom eq "-"))) { 
        die "ERROR problem parsing, you specified --fmt2 but tblout file appears to have NOT been created with --fmt 2, retry without --fmt2\nproblematic line:\n$line\n"; 
      }
      if((! $do_fmt2) && (($pass eq "+") || ($pass eq "-"))) { 
        die "ERROR problem parsing, you did not specify --fmt2 but tblout file appears to have been created with --fmt 2, retry with --fmt2\nproblematic line:\n$line\n"; 
      }
      die "ERROR unable to parse, problematic line:\n$line\n";
    }
    if(($do_minscore) && ($score < $minscore)) { 
      ; # skip
    }
    elsif(($do_maxevalue) && ($evalue > $maxevalue)) { 
      ; # skip
    }
    else { 
      my $attributes = "evalue=" . $evalue; # default to just evalue
      if($do_all_attributes) { 
        if($do_fmt2) { 
          $attributes .= sprintf(";idx=$idx;seqaccn=$seqaccn;mdlaccn=$mdlaccn;clan=$clan;mdl=$mdl;mdlfrom=$mdlfrom;mdlto=$mdlto;trunc=$trunc;pass=$pass;gc=$gc;bias=$bias;inc=$inc;olp=$olp;anyidx=$anyidx;anyfrct1=$anyfrct1;anyfrct2=$anyfrct2;winidx=$winidx;winfrct1=$winfrct1;winfrct2=$winfrct2;%s$desc", ($do_hidedesc ? "" : "desc="));
        }
        else { 
          $attributes .= sprintf(";seqaccn=$seqaccn;mdlaccn=$mdlaccn;mdl=$mdl;mdlfrom=$mdlfrom;mdlto=$mdlto;trunc=$trunc;pass=$pass;gc=$gc;bias=$bias;inc=$inc;%s$desc", ($do_hidedesc ? "" : "desc="));
        }
      }
      elsif($do_no_attributes) { 
        $attributes = "-";
      }
      elsif($do_de_attributes) { 
        $attributes = $desc;
      }
      if(defined $extra) { 
        if($attributes eq "-")       { $attributes = ""; }
        elsif($attributes !~ m/\;$/) { $attributes .= ";"; }
        $attributes .= $extra . ";";
      }
      printf("%s\t%s\t%s\t%d\t%d\t%.1f\t%s\t%s\t%s\n", 
             $seqname,                             # token 1: 'sequence' (sequence name)
             $source,                              # token 2: 'source'
             $mdlname,                             # token 3: 'feature' (model name) you may want to change this to 'ncRNA'
             ($strand eq "+") ? $seqfrom : $seqto, # token 4: 'start' in coordinate space [1..seqlen], must be <= 'end'
             ($strand eq "+") ? $seqto : $seqfrom, # token 5: 'end' in coordinate space [1..seqlen], must be >= 'start'
             $score,                               # token 6: 'score' bit score
             $strand,                              # token 7: 'strand' ('+' or '-')
             ".",                                  # token 8: 'phase' irrelevant for noncoding RNAs
             $attributes);                         # token 9: attributes, currently only E-value, unless --all, --none or --desc
    }
  }
}
```

3. 运行
`perl infernal-tblout2gff.pl --cmscan --fmt2 genome.tblout >genome.infernal.ncRNA.gff3`

输入是Infernal的table格式输出文件，输出是gff3格式的ncRNA注释文件。

