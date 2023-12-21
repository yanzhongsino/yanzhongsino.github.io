---
title: 线粒体的重复介导重组的鉴定和计算重组频率
date: 2023-03-14
categories: 
- omics
- organelle
- recombination
tags: 
- mitogenome
- repeat
- recombination
- rearrangement
- insert size
- recombination frequency
- ROUSFinder
description: 线粒体的重复介导重组的鉴定和计算重组频率。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105213&auto=1&height=32"></iframe></div>

# 1. 线粒体的重复介导重组（repeat-mediated recombination）
1. 线粒体的重复序列
- 通常在植物线粒体上存在重复序列（这里的重复序列不是核基因组的TE等转座元件的概念，就是一段序列在另一个位置同时存在完全一致的序列），则有可能介导重组，从而生成线粒体的多种构象。重复序列常常成对存在（也有一些超过两个），被称作重复对（repeat pairs）。
2. 鉴定的背景
- Illumina PE测序是双端测序，测序获得的reads常见的长度是150bp。一对reads的联合体的长度被称为插入尺寸（insert size）。插入尺寸（insert size）常在350bp左右，但具体的每对测序reads都不一样，要通过把reads进行mapping到参考序列上，才能测量insert size。
- 如果是repeat pairs的长度小于测序reads的insert size长度，可以构建两种构象（参考构象和重组构象），把reads mapping到两种构象来判断repeat pairs是否介导重组（如果两种构象都有mapped reads则是repeat pair介导重组的证据），并且通过mapped reads的计数来计算重组频率（recombination frequency）。

# 2. 鉴定重复
## 2.1. 鉴定重复的脚本
这篇文章Repeats of Unusual Size in Plant Mitochondrial Genomes: Identification, Incidence and Evolution： https://academic.oup.com/g3journal/article/9/2/549/6026745 总结了植物线粒体基因组的重复序列的特征。研究表明，植物线粒体基因组比动物的要大，包含大量的非编码DNA，突变率低，重排率高。

- 文章提供了python脚本ROUSFinder.py，用在鉴定线粒体的重复序列上。
- 脚本是python2写的，依赖主要有blastn。
- 有以下三个版本。

1. ROUSFinder.py
- 调用blastn进行一条序列与自身的成对比对。
- 默认参数是最小重复50bp，E value 10,000， match的赏分是+1，mismatch的罚分是-20。
- 脚本贴在下面：

```python
#! /usr/bin/env python  
import sys, math, os, argparse, csv  
csv.field_size_limit(sys.maxsize)  
  
# January 16, 2018 version 1.1  
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
# Percent identity is limited to >=99%, to allow for sequencing errors of <1%.  
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
  
parser = argparse.ArgumentParser(description='Find repeats in a fasta sequence file')  
parser.add_argument('infile', action='store', help='Input .fasta file')  
parser.add_argument('-o', action='store', dest='outfile', help='Output file name seed, default is input_repeats', default='default')  
parser.add_argument('-m', action='store', dest='minlen', help='Minimum length of matches to keep, default=24', default='24')  
parser.add_argument('-b', action='store', dest='blast_path', help='Path to blastn program, default is /usr/bin/', default='/usr/bin/')  
parser.add_argument('-k', action='store_true', dest='keep', help='True to keep temp files', default=False)  
parser.add_argument('-gb', action='store_true', dest='genbank', help='True to write GenBank format file', default=False)  
results = parser.parse_args()  
infile = results.infile  
outfile = results.outfile  
minlen = int(results.minlen)  
blast_path = results.blast_path  
keep = results.keep  
genbank = results.genbank  
  
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
os.system(blast_path+'blastn -query '+infile+' -strand plus -subject '+infile+' -word_size '+wordsize+' -reward 1 -penalty -20 -ungapped -dust no -soft_masking false -evalue 1000 -outfmt "10 qstart qend length sstart send mismatch sstrand qseq" | tail -n+2 > tempblast1.txt')  
os.system(blast_path+'blastn -query '+infile+' -strand minus -subject '+infile+' -word_size '+wordsize+' -reward 1 -penalty -20 -ungapped -dust no -soft_masking false -evalue 1000 -outfmt "10 qstart qend length sstart send mismatch sstrand qseq" > tempblast2.txt')  
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
rc.write('Sequence\tGenome_size\tNumRepeats\tAvgSize\tAvgCopyNum\n')  
  
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

2. MultipleRepeats.py
- MultipleRepeats.py批量运行一个文件夹下的每个序列文件。
- 脚本贴在下面：

```python
#! /usr/bin/env python  
import sys, math, os, argparse  
  
# Usage: -din directory of files to find repeats in  
#        -word word_size  
  
parser = argparse.ArgumentParser(description='Find repeats in a directory of fasta sequence files')  
parser.add_argument('-din', action='store', dest='din', help='Input .fasta directory')  
parser.add_argument('-word', action='store', dest='word', help='Word size for blast')  
results = parser.parse_args()  
din = results.din  
word = results.word  
  
li = os.listdir(din)  
inputs = filter(lambda x: '.fasta' in x, li)  
inputs.sort()  
  
for i in range(len(inputs)):  
    infile = str(inputs[i])  
    os.system("/home/alan/applications/ROUSFinder.py -m "+word+" "+din+infile) 
```

3. ROUSFinder2.py
- ROUSFinder2.py可以在命令行设置match赏分和mismatch罚分。
- 脚本贴在下面：

```python
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

## 2.2. 运行脚本鉴定重复
选择ROUSFinder2.py脚本来鉴定线粒体的重复序列。
1. 脚本参数
- 必需参数：fasta格式的输入序列
- -o out：输出文件名称
- -m 50：鉴定的重复序列的最小长度，默认是50bp。
- -b /path/to/blastn/：blastn命令的路径，默认是/usr/bin/。
- -k：保留临时文件，out_tempblast.txt和out_temprepeats.txt
- -gb：生成重复序列的genbank格式文件，out_repeats.gb.txt
- -rew 1：match赏分，默认是+1
- -pen 20：mismatch罚分，默认是-20
2. 运行脚本
- `ROUSFinder2.py input.fa -m 30 -gb -o out`

## 2.3. 结果文件
1. out_rep_table.txt：包含每个repeat单元的名称，长度，起始位置和终止位置，正负链（plus或minus）。
- 第一行显示了第一个染色体的名称和长度。
- 后面的行有五列，分别是repeat名称，repeat长度，repeat在染色体序列上的起始位置和终止位置，repeat的正负链方向。
- 通常前面只有一行信息的重复单元（如Repeat_1和Repeat_2）需要手动去除，留下后面有2个或以上重复单元的信息（如Repeat_3和Repeat_4）作为重复序列鉴定的结果。

```
Chr01	147501
Repeat_1	2702	1	2702	plus
Repeat_2	2480	1	2480	plus
Repeat_3	1171	1933	3103	plus
Repeat_3	1171	2322	3492	plus
Repeat_4	938	1696	2633	plus
Repeat_4	938	2237	3174	plus
```

2. out_rep.fasta：包含fasta格式的repeat序列
3. out_rep_counts.txt：包含repeat的总数（NumROUS），平均长度（AvgSize），平均数量（AvgCopyNum），和每个repeat的长度和数量。
4. out_repeats.gb.txt：使用-gb参数则生成此文件，是genbank格式的repeat的注释文件。
5. out_binned.txt：包含>50bp,100bp,150bp,...,1000bp,1250bp,1500bp,...,10000bp的repeat的数量
6. out_tempblast.txt：使用-k参数会保留两个临时temp文件
7. out_temprepeats.txt：使用-k参数会保留两个临时temp文件。
- out_temprepeats.txt是用重复单元与输入序列做blastn的-outfmt 10的结果，包含qseqid length sstart send sstrand qcovhsp六列内容。
- out_rep_table.txt结果文件是从这个out_temprepeats.txt临时文件中提取生成的。

## 2.4. 修改鉴定重复的脚本，使其显示重复序列所在染色体位置
ROUSFinder2.py脚本鉴定线粒体的重复序列是被设计的用于鉴定一条序列与自身的重复，所以结果文件中不包含重复所在染色体位置信息。好在脚本容易修改，就修改了脚本使其显示染色体位置，可用于多染色体线粒体基因组的重复序列鉴定。

使用修改后脚本的输出结果中两个文件（out_rep_table.txt和out_temprepeats.txt）有所不同，其他结果文件都与修改前的脚本的结果一致。

1. 修改后的脚本ROUSFinder2.0_adjusted.by.yz.py
- ROUSFinder2.0_adjusted.by.yz.py可以在结果文件out_rep_table.txt中显示染色体位置信息。
- 修改后脚本保存在：https://github.com/yanzhongsino/bioscripts/blob/main/modified_scripts/ROUSFinder2.0_adjusted.by.yz.py
- 修改后脚本也贴在下面：

```python
#! /usr/bin/env python  
import sys, math, os, argparse, csv  
csv.field_size_limit(sys.maxsize)  

# minior adjustments by Yan Zhong, November 27, 2023  
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
os.system(blast_path+'blastn -query '+outfa+' -strand both -subject '+infile+' -word_size '+wordsize+' -reward 1 -penalty -20 -ungapped -dust no -soft_masking false -evalue 1000 -outfmt "10 qseqid length sstart send sstrand qcovhsp sseqid qstart qend" > '+temprepeats)  # adjusted by yz
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
# rt.write(seqname+'\t'+str(seqlen)+'\n') # adjusted by yz
rt.write('repeat.name'+'\t'+'alignment.length'+'\t'+'repeat.start'+'\t'+'repeat.end'+'\t'+'strand'+'\t'+'qcovhsp'+'\t'+'chr.name'+'\t'+'chr.start'+'\t'+'chr.end'+'\n')  # adjusted by yz
templist = []  
gblist =[]  
  
# look at each repeat in turn  
for i in range(len(replist)):  
    # if repeat is good (>98% identical to another one), write it to the file, and put the name in a list  
    if int(replist[i][5])>98:  
        rt.write(str(replist[i][0])+'\t'+str(replist[i][1])+'\t'+str(replist[i][7])+'\t'+str(replist[i][8])+'\t'+str(replist[i][4])+'\t'+str(replist[i][5])+'\t'+str(replist[i][6])+'\t'+str(replist[i][2])+'\t'+str(replist[i][3])+'\n')  # adjusted by yz
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

2. 修改前脚本的结果文件out_rep_table.txt
- 第一行显示了第一个染色体的名称和长度。
- 后面的行有五列，分别是重复单元的名称(repeat.name)，重复单元的长度(alignment.length)，重复单元在输入的染色体序列上的起始位置和终止位置(chr.start和chr.end)，重复单元的正负链方向(strand)。

```
Chr01	147501
Repeat_1	2702	1	2702	plus
Repeat_2	2480	1	2480	plus
Repeat_3	1171	1933	3103	plus
Repeat_3	1171	2322	3492	plus
Repeat_4	938	1696	2633	plus
Repeat_4	938	2237	3174	plus
```

3. 修改后脚本的结果文件out_rep_table.txt
- 添加了标题行。包含重复单元的名称(repeat.name)，重复单元的长度(alignment.length)，重复单元在重复序列上起始和终止位置(repeat.start和repeat.end)，重复单元的正负链方向(strand)，每个高度相似片段HSP中的重复单元的覆盖度(Query Coverage Per HSP, qcovhsp)，染色体序列名称(chr.name)，重复单元在输入的染色体序列上的起始位置和终止位置(chr.start和chr.end)。
- 添加了列的展示，主要是染色体序列名称(chr.name)，还包括repeat.start和repeat.end，qcovhsp。
- 调整了列的展示顺序。

```
repeat.name	alignment.length	repeat.start	repeat.end	strand	qcovhsp	chr.name	chr.start	chr.end
Repeat_1	2702	1	2702	plus	100	Chr38	1	2702
Repeat_2	2480	1	2480	plus	100	Chr39	1	2480
Repeat_3	1171	1	1171	plus	100	Chr34	1933	3103
Repeat_3	1171	1	1171	plus	100	Chr09	2322	3492
Repeat_4	938	1	938	plus	100	Chr38	1696	2633
Repeat_4	938	1	938	plus	100	Chr32	2237	3174
```

4. 修改后脚本的结果文件out_temprepeats.txt
- 修改后脚本的结果文件out_temprepeats.txt比起修改前在后面多加了三列信息chr.name,repeat.start和repeat.end。

5. notes（与脚本的修改无关）
- 从修改后脚本的结果文件out_rep_table.txt的示例文件可看出，qcovhsp列绝大部分情况下是100，因为这个值高于98才会被鉴定为重复单元。
- repeat.start和repeat.end两列的值也一般是固定的，从1到重复单元的末位。这两列值是用重复序列与输入序列做blast，比对结果中比对到的重复序列的重复单元的位置，通常只有完全比对上（repeat.start和repeat.end两列值从1到重复单元的末位）的才被鉴定为重复单元。

## 2.5. 重复的应用
1. 辅助线粒体组装
- 通过鉴定出来的重复，可以帮助线粒体组装得更完整。
- 调整线粒体的已有组装。比如通过计算重组率，把主导的优势构象作为最后的线粒体组装构象。
2. 鉴定重组
- 重复序列可能介导重组，所以鉴定重复是鉴定重组的基础。

# 3. 鉴定重组和计算重组率
鉴定出线粒体的重复序列后，还可以进一步鉴定这些重复序列是否介导了重组。

重组会导致构象的变化，构象变化的形式包括：
1. 位于不同染色体环上的一对重复序列可能介导重组，使得两个小环变成一个大环。
2. 位于同一个染色体环上的同方向的一对重复序列可能介导重组，使得一个大环变成两个小环。
3. 位于同一个染色体环上的反方向的一对重复序列，可能在其他重复对的重组变换下变成同方向或者两个小环的状态。

## 3.1. 鉴定重组的步骤
### 3.1.1. 构建参考构象和重组构象的序列
1. 针对一对可能造成重组的重复序列，构建参考构象和重组构象
2. 在参考构象和重组构象中，以重复序列加上两翼各300bp作为参考构象序列(ref_1和ref_2)和重组构象序列(rec_1和rec_2)
3. 构建参考序列和mapping注意事项
- 可以把参考构象序列和重组构象序列（一共四条）一同作为参考序列（reference）用于mapping，避免同一条reads mapping到多个构象序列从而重复计数的情况。
- 如果担心叶绿体派生的reads影响重组率的计算，还可以加上叶绿体基因组作为参考序列（reference），这样叶绿体的reads会被mapping到叶绿体上，从而起到过滤的作用。
- 如果担心核基因派生的reads影响，可以用一个cutoff值(比如100bp)，小于100bp的mapping被筛除。

### 3.1.2. 把reads往参考序列进行mapping
1. 把reads往参考序列进行mapping，之后再根据mapping的reads的位置进行筛选，筛选出横跨repeat区域的有效mapping。
2. 比如repeat长度为220bp，两边各截取300bp，共820bp长度的参考序列。这里把跨越220bp的repeat，并且两边各至少映射上10bp的reads作为有效mapping。这意味着，mapping的起始位置需要小于290，终止位置需要大于530bp。
3. bwa进行mapping后得到的bam文件的第4列代表read比对到的参考序列最左侧的位置坐标（未必对上第4列为0）；第9列代表pair read完全匹配到同一条参考序列时，两个read之间的长度。可以理解为insert size。

```shell
a = 530 # 设置终止位置需要大于的值，这里是530bp
bwa index reference.fa # 为参考序列建立索引
bwa mem -t 4 reference.fa  sample_1.clean.fq sample_2.clean.fq | samtools sort -@ 4 -m 4G > reference.bam # 映射reads到参考序列
samtools view reference.bam |awk -F"\t" -v awka="$a" '$4<290 && ($4+$9)>awka {print $0}' > reference.bam.filter # 筛选起始位置小于290，终止位置大于530bp的mapped reads。这里的`awk -F"\t"`参数设定列分隔符非常重要，已经坑过我两把了。
```

### 3.1.3. 计算重组率(recombination rate)
1. 把reads往参考序列进行mapping得到有效映射的比对文件reference.bam.filter后，计算不同参考序列的有效映射的reads数量（即数据行数）。
2. 重组构象的两条序列(rec_1和rec_2)的比对reads数量之和作为分子，重组构象的两条序列(rec_1和rec_2)的比对reads数量与参考构象的两条序列(ref_1和ref_2)的比对reads数量之和作为分母，两者的比值可作为重组率。

```shell
n_ref_1 = $(cat reference.bam.filter | awk '$3 == "ref_1" {print $0}' |wc -l) # 统计参考构象序列ref_1上有效映射reads的数量
n_ref_2 = $(cat reference.bam.filter | awk '$3 == "ref_2" {print $0}' |wc -l) # 统计参考构象序列ref_2上有效映射reads的数量
n_rec_1 = $(cat reference.bam.filter | awk '$3 == "rec_1" {print $0}' |wc -l) # 统计参考构象序列rec_1上有效映射reads的数量
n_rec_2 = $(cat reference.bam.filter | awk '$3 == "rec_2" {print $0}' |wc -l) # 统计参考构象序列rec_2上有效映射reads的数量
mapping_rate = $(($n_rec_1+$n_rec_2)/($n_ref_1+$n_ref_2+$n_rec_1+$n_rec_2)) # 计算重组率
```

# 4. references
1. ROUSFinder.py脚本：https://academic.oup.com/g3journal/article/9/2/549/6026745


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=30% title="wechat_public_QRcode.png" align=center/>