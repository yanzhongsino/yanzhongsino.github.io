---
title: linux系统的压缩/解压缩和打包/解包工具
date: 2024-06-18
categories:
- linux
- command
- compress
tags:
- linux
- compress
- gzip
- pigz
- bigz2
- tar
- compress
- zip
- rar

description: linux系统下的压缩工具，以及压缩和解压缩的操作，打包工具，打包和解包操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1485931&auto=1&height=32"></iframe></div>

**一句话总结**

linux系统下最常见的压缩工具是**gzip（压缩文件格式gz）**，可用**多线程命令pigz**替代。另一个压缩效率更高的常用工具是**bgzip2（压缩文件格式bz2）**。其余的compress（压缩文件格式Z），zip（压缩文件格式zip），rar（压缩文件格式rar）都不常用。另外**打包工具tar**也常常调用压缩工具进行打包和压缩。

# 1. linux系统下的压缩和打包命令快查表

<table>
	<caption><h4>压缩和打包命令快查表</h4></caption>
	<thead>
	<tr>
		<th>文件格式</th>
		<th>工具</th>
		<th>压缩/打包命令</th>
		<th>解压/解包命令</th>
	</tr>
	</thead>
	<tbody>
	<tr>
		<td>file.gz</td>
		<td>gzip和gunzip</td>
		<td>gzip file</td>
		<td>gzip -d file.gz或gunzip file.gz</td>
	</tr>
	<tr>
		<td>file.gz</td>
		<td>pigz（多线程的gzip）</td>
		<td>pigz -p 8 file（用8个线程压缩）</td>
		<td>pigz -p 8 -d file.gz（用8个线程解压缩）</td>
	</tr>
	<tr>
		<td>file.tar</td>
		<td>tar（打包/解包）</td>
		<td>tar cvf file.tar *.jpg (把所有jpg文件打包进tar包，命名为file.tar文件)</td>
		<td>tar xvf file.tar</td>
	</tr>
	<tr>
		<td>file.tar.gz或file.tgz</td>
		<td>tar（打包/解包的同时压缩/解压缩）</td>
		<td>tar zcvf file.tar.gz *.jpg (把所有jpg文件打包压缩进file.tar.gz文件)</td>
		<td>tar zxvf file.tar.gz</td>
	</tr>
	<tr>
		<td>file.bz2</td>
		<td>bzip2和bunzip2</td>
		<td>bzip2 file</td>
		<td>bzip2 -d file.bz2或bunzip2 file.bz2</td>
	</tr>
	<tr>
		<td>file.Z</td>
		<td>compress和uncompress</td>
		<td>compress file</td>
		<td>uncompress file.Z</td>
	</tr>
	<tr>
		<td>file.zip</td>
		<td>zip和unzip</td>
		<td>zip file.zip *.jpg (把所有jpg文件压缩成一个zip包)</td>
		<td>unzip file.zip</td>
	</tr>
	<tr>
		<td>file.rar</td>
		<td>rar和unrar （需要下载安装RAR for Linux工具）</td>
		<td>rar a file *.jpg （把所有jpg文件压缩成一个rar包，名为file.rar）</td>
		<td>unrar e file.rar （把file.rar中所有文件解压出来）；unrar x file.rar （把file.rar中所有文件解压出来，并按目录层级）</td>
	</tr>
	</tbody>
</table>

# 2. linux系统下的压缩工具
## 2.1. gzip工具（.gz压缩格式）
1. gzip
- gzip是linux系统自带的压缩工具，生成压缩文件的后缀为gz（linux最常见的压缩格式）。
- gunzip是gzip的硬连接，与gzip命令几乎没有区别，主要用于解压缩gz格式文件。
- pigz是gzip的多线程并行版本，完全兼容gzip，可以直接替代gzip。
- gz格式常用于压缩生物序列文件，如fq，fa文件。绝大多数生物信息学软件都支持直接读取压缩后的fq.gz和fa.gz文件。
2. 基础用法
- `gzip file.txt`：压缩file.txt文件。gzip默认生成与原始文件同前缀的压缩后文件file.txt.gz，并删除原始文件file.txt。
- `gzip -k file.txt`：压缩file.txt文件。默认生成与原始文件同前缀的压缩后文件file.txt.gz，`-k`参数不删除原始文件file.txt。
- `gzip -c file.txt > sample.gz`：指定压缩文件名为sample.gz。此命令把file.txt压缩到sample.gz，并保留原始文件file.txt。`-c`参数把压缩结果输出到标准输出，并保留原始文件，不会默认生成与原始文件同名压缩文件。
- `gzip -d file.txt.gz`：解压缩file.txt.gz文件。此命令会生成解压缩后文件file.txt，并删除原始文件file.txt.gz。
- `gzip -r Directory`：压缩Directory目录，使用-r参数递归压缩目录中所有文件。
3. 进阶用法
- `gzip file1.txt file2.txt file3.txt`：分别压缩多个文件。此命令会生成压缩后文件file1.txt.gz，file2.txt.gz，file3.txt.gz，并删除原始文件file1.txt，file2.txt，file3.txt。
- `cat file1.txt file2.txt | gzip > file.gz`或者`gzip -c file1.txt file2.txt > file.gz`：gzip用于通道。此命令合并多个文件，并压缩合并后文件。如果gzip后不跟文件或者跟着`-`代表文件，则读取标准输入作为压缩对象。
- `tar cf - Directory | gzip |ssh user@remotehost "cat > directory.tar.gz"`：结合gzip和ssh进行远程备份
- `cat newdata.txt | gzip >> existingdata.gz`：用gzip压缩新的数据并追加到现有压缩文件。
4. 参数
- -c, --stdout：write on standard output, keep original files unchanged. 结果写到标准输出，原始文件保持不变。想要自行指定压缩文件名或者在管道中使用gzip则需要用这个参数。
- -d：decompress，解压缩。
- -1, -9：压缩级别1到9里面调整，-1压缩快但压缩程度低，-9压缩慢但压缩程度高。默认-6. 
- -f：force overwrite of output file and compress links. 默认情况下，如果压缩文件已存在，gzip不会覆盖它。-f参数强制覆盖已有压缩文件，进行强制压缩。
- -k：keep (don't delete) input files。默认会删除原始文件，-k参数则保留原始文件。
- -r：压缩目录，递归地压缩目录中的所有文件。
- -h：give this help。帮助文档。
- -t：test compressed file integrity. 测试压缩文件的完整性，而不解压。`gzip -t file.txt.gz`，如果输出显示OK，则表示文件完整无损。
- -l：list compressed file contents。列出压缩文件的内容，而不解压。
- -L：display software license
- -n：do not save or restore the original name and timestamp. 压缩文件时不保存原始文件名和时间戳。
- -N：save or restore the original name and timestamp
- -q：quiet，suppress all warnings
- -S：指定压缩文件的后缀。
- -v, --verbose：以冗长模式输出，会显示每个文件的文件名和压缩率。
- -V, --version：列出软件版本号。

## 2.2. pigz (Parallel Implementation of GZip) ：多线程的gzip
1. pigz可以直接替代gzip
- 因为gzip只支持单线程运行，所以推荐使用gzip的多线程并行版本pigz，完全兼容gzip。
- pigz支持gzip的所有参数，可以完美替代gzip，使压缩更高效。
- pigz除了支持gzip的参数外，还要自己特有的参数。
2. pigz的特有参数
- -p n：指定使用n个线程，默认8线程。
- -0 to -9, -11：压缩级别，除了gzip的1-9，还支持-11的极致压缩。
- -b 128K：指定压缩block size，默认是128K。
- -i, --independent    Compress blocks independently for damage recovery
- -I, --iterations n   Number of iterations for -11 optimization
- -J, --maxsplits n    Maximum number of split blocks for -11
- -Y  --synchronous    Force output file write to permanent storage
- -K, --zip            Compress to PKWare zip (.zip) single entry format
- -z, --zlib           Compress to zlib (.zz) instead of gzip format
- --                   All arguments after "--" are treated as files
- -O  --oneblock       Do not split into smaller blocks for -11
3. 使用pigz注意事项
- 当使用多线程时，pigz可能会消耗大量的CPU资源，因此在负载较重的系统上使用时需谨慎。
- 请确保在使用多线程选项时，您的系统具有足够的处理器核心。
- 压缩大文件时，pigz的内存消耗可能会增加。
- 确保在脚本中使用时正确处理了pigz的退出状态，以便于捕获任何可能的错误。

## 2.3. bzip2（.bz2格式）
1. bzip2
- bzip2是压缩程度更高，但压缩速度更慢的工具，生成的压缩文件后缀为bz2。
2. 参数
- -h --help           print this message
- -d --decompress     force decompression. 解压缩
- -z --compress       force compression. 强制压缩
- -k --keep           keep (don't delete) input files. 保留原始文件
- -f --force          overwrite existing output files. 覆盖已存在的输出文件
- -t --test           test compressed file integrity. 测试压缩文件的完整性
- -c --stdout         output to standard out. 输出结果到标准输出
- -q --quiet          suppress noncritical error messages
- -v --verbose        be verbose (a 2nd -v gives more)
- -L --license        display software version & license
- -V --version        display software version & license
- -s --small          use less memory (at most 2500k). 压缩过程使用更小的内存。
- -1 .. -9            set block size to 100k .. 900k. 压缩级别1到9里面调整，-1压缩快但压缩程度低，-9压缩慢但压缩程度高。

## 2.4. compress
1. compress
- compress命令生成的压缩文件后缀是Z。
2. 参数
- -d   If given, decompression is done instead. 解压缩。
- -c   Write output on stdout, don't remove original.
- -b   Parameter limits the max number of bits/code. 压缩效率，9-16之间的数值，数值越大，压缩效率越高。默认是16。
- -f   Forces output file to be generated, even if one already exists, and even if no space is saved by compressing. If -f is not used, the user will be prompted if stdin is a tty, otherwise, the output file will not be overwritten. 强制覆盖已有文件。
- -v   Write compression statistics. 显示命令执行过程。
- -V   Output vesion and compile options. 显示命令版本及默认参数。
- -r   Recursive. If a filename is a directory, descend into it and compress everything in it. 压缩目录时递归压缩。

# 3. 打包工具：tar
## 3.1. 压缩和打包的区别
- 打包是指将一大堆文件或目录变成一个总的文件。
- 压缩则是将一个大的文件通过一些压缩算法变成一个小文件。
- Linux中很多压缩程序（如gzip bzip2命令）只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip bzip2命令）。

## 3.2. 打包工具：tar（tape archive）
- tar命令可以把一大堆的文件和目录全部打包成一个文件，这对于备份文件或将几个文件组合成为一个文件以便于网络传输是非常有用的。

## 3.3. 参数
1. 常用参数
- -f：filename（文件名），这个参数后只能接文件名，所以一定要放在最后一个参数。
- -c, --create：创建一个新归档（打包）
- -x, --extract, --get：从归档中解出文件（解包）
- -A, --catenate, --concatenate：追加 tar 文件至归档
- -r, --append：追加文件至归档结尾
- -u, --update：仅追加比归档中副本更新的文件
- -d, --diff, --compare：找出归档和文件系统的差异；--delete：从归档(非磁带！)中删除
- -t, --list：列出归档内容；--test-label：测试归档卷标并退出
- -k, --keep-old-files：don't replace existing files when extracting, treat them as errors. 保留原始文件。
- -O, --to-stdout：解压文件至标准输出
- -C, --directory=DIR：解压文件至目录 DIR
- -v, --verbose：详细地列出处理的文件
2. 压缩参数
- -a, --auto-compress：使用归档后缀名来决定压缩程序
- -I, --use-compress-program=PROG：通过 PROG 过滤(必须是能接受 -d选项的程序)
- -z, --gzip, --gunzip, --ungzip：通过 gzip 过滤归档
- -Z, --compress, --uncompress：通过 compress 过滤归档
- -j, --bzip2：通过 bzip2 过滤归档
- -J, --xz：通过 xz 过滤归档
3. 其他参数
- -p：保留原本文件的属性，如权限。
- --strip-components=2：解压时从文件名中清楚2个引导部分。

## 3.4. 实例

```shell
tar -cf files.tar file1.txt file2.txt # 打包file1.txt file2.txt到文件files.tar。仅打包不压缩。
tar -xf files.tar # 解包不解压。

tar -zcf files.tar.gz file1.txt file2.txt  # 打包并调用gzip压缩file1.txt file2.txt到文件files.tar.gz
tar -zxf files.tar.gz # 解包并调用gzip解压缩

tar -jcf files.tar.bz2 file1.txt file2.txt # 打包并调用bzip2压缩file1.txt file2.txt到文件files.tar.bz2
tar -jxf files.tar.bz2 # 解包并调用bzip2解压

tar -tf files.tar # 列出打包文件 files.tar 中的所有文件，但不解包。-t是列出文件的意思。
tar -ztf files.tar.gz # 列出打包压缩文件 files.tar.gz 中的所有文件，但不解压缩也不解包。gz文件需要加上-z参数调用gzip。
tar -zxf files.tar.gz file1.txt # 只将files.tar.gz内的部分文件（如file1.txt）解压出来

tar -rf all.tar *.gif # 这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。
tar -uf all.tar logo.gif # 这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。
```

- 上面的实例通常都会加上-v参数详细列出处理文件。


# 4. references
1. 压缩命令：https://www.runoob.com/w3cnote/linux-tar-gz.html
2. pigz的用法实例：https://bashcommandnotfound.cn/article/linux-pigz-command#:~:text=Pigz%E6%98%AF%E4%B8%80%E4%B8%AAgzip%E5%8E%8B%E7%BC%A9,%E7%9A%84%E4%B8%80%E4%B8%AA%E9%AB%98%E6%95%88%E6%9B%BF%E4%BB%A3%E5%93%81%E3%80%82
3. tar用法：https://wangchujiang.com/linux-command/c/tar.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>