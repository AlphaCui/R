## SRA-toolkits 的简介

> <https://trace.ncbi.nlm.nih.gov/Traces/sra/sra.cgi?view=toolkit_doc>

​	用来下载 SRA runs



## 常用工具

`fastq-dump`：Convert SRA data into fastq format

`prefech`：Allows command-line downloading of SRA, dbGaP, and ADSP data

`sam-dump`: Convert SRA data to sam format

`sar-pileup`:  Generate pileup statistics on aligned SRA data

> 生成对齐的SRA数据的堆积统计信息





## SRA-toolkits 的安装

​	使用 bioconda 安装 sra-toolkits

```shell 
conda search sra
# 根据出现的错误到网站上找到这个包
# 根据包的安装方式进行安装
conda installconda install -c bioconda sra-tools
```



## 数据下载

```shell
prefetch SRRXXXX
```

- 注意文件如果较大，使用-X 调整允许最大文件

	```shell 
	prefetch -X 100G SRR1197490
	```

	

	- 安装aspera后，加上参数`-a` 调用aspera来下载 还是 -v 来调用？？？

- 如果需要同时下载多个srr数据，将srr号写入txt文件，然后写一个for-loop脚本来执行

## 格式转换

SRAtoolkit 格式转换 SRA --> fastq

- 使用 `fastq-dump`  也可以使用 `fasterq-dump`

- `-O|--outdir` : 输出到指定文件夹

- 单端：`fastq-dump  SRR6470965.sra`     结果生成 ：SRR6470965.fastq

- **双端**：`fastq-dump --split-3  DRR002018.sra`     结果生成   DRR002018_1.fastq；DRR002018_2.fastq

	- `--split-3` : 将双端测序分为两份,放在不同的文件,但是对于一方有而一方没有的reads会单独放在一个文件夹里

	> 对于一个你不知道到底是单端还是双端的SRA文件,一律用`--split-3`

- 如果想把一个文件夹里的所有的sra格式都转换了，使用循环

```shell
for  i in ls *.sra;
do fastq-dump --spit-3 $i;
done
```

### fastq-dump

> <https://www.jianshu.com/p/a8d70b66794c>

- `--split-spot`: 将双端测序分为两份,但是都放在同一个文件中
- `--split-files`: 将双端测序分为两份,放在不同的文件,但是对于一方有而一方没有的reads直接丢弃
- `--split-3`    : 将双端测序分为两份,放在不同的文件,但是对于一方有而一方没有的reads会单独放在一个文件夹里



#### fasterq-dump

> <https://www.jianshu.com/p/5c97a34cc1ad>

- `fasterq-dump`的用法和`fastq-dump`一样，如下所示

```shell
fasterq-dump --split-3 SRR5318040.sra 
```

- 重点参数是`-e|threads`, 用于选择使用多少线程进行运行，默认是6个线程。 同时考虑到有些人容易着急，还提供了`-p`选项用于显示当前进度。

```shell
time fastq-dump --split-3 -O test SRR5318040.sra
# 558.76s user 41.36s system 101% cpu 9:51.82 total
time fasterq-dump --split-3 SRR5318040.sra -e 20 -o SRR5318040
# 582.70s user 121.06s system 1130% cpu 1:02.25 total
```



## fastqc 质控

> https://zhuanlan.zhihu.com/p/20731723

- 参数解析

	> -o --outdir:输出路径
	> --extract：结果文件解压缩
	> --noextract：结果文件压缩
	> -f --format:输入文件格式.支持bam,sam,fastq文件格式
	> -t --threads:线程数
	> -c --contaminants：制定污染序列。文件格式 name[tab]sequence
	> -a --adapters：指定接头序列。文件格式name[tab]sequence
	> -k --kmers：指定kmers长度（2-10bp,默认7bp）
	> -q --quiet： 安静模式

- 运行命令

	```shell
	fastqc -t 8 -o path/fastqc sample1_R1.fq sample1_R2.fq
	```



- **结果查看**

	> https://www.jianshu.com/p/835fd925d6ee>

	> https://zhuanlan.zhihu.com/p/20731723

- 产生两个结果文件：

	- html：网页版结果	

- zip：本地结果压缩文件

- 需要重点关注的结果：

	- Basic Statistics：对数据量的概览

	- Per base sequence quality：reads每个位置测序质量最直接的展示

	- Per sequence quality scores：总体reads测序质量趋势

		- Per base sequence content：ATGC含量估计测序是否存在偏差

		- Sequence Duplication Levels]：影响测序的因素太多，查看是否存在污染，数据处理时是否需要去冗余；现在数据量都可以满足需求，因此前期数据处理时，尽量高标准，严格质控；

			

## cutadate











# 实战代码

```shell
## 下载数据
prefetch SRR5442625   # 下次考虑加上参数 -a，使用 aspera 来下载
prefetch SRR5442617

# 格式转换
> for i in *sra;
> do fasterq-dump --split-3 -O ./fast_results/ $i;
> done
## 两个 SRA 文件，得到四个 fastq 文件，都是双端测序


# 质控查看

## 编写脚本
#! /bin/bash
for i in *fastq;
do fastqc -o ./fastqc_results/ $i;
done

## 执行上一个脚本，并将日志输出
nohup bash fastqc.sh > fastqc_log 2>&1 &


```

```shell
## 对文件进行重命名
### 将 SRRXXXXX.sra_1.fastq 转变为 SRRXXXXX_1.fastq
#### 如何用 rename 进行批量重命名？？？？

# 脚本
#！ /bin/bash
for i in *fastq;
do
name_1=${i:0:10}     # 根据索引位置从左侧开始提取字符串,从 0 开始计
name_1_1=${i%sra*}   # 把 sra 左边的字符提取出来 string%chars*

#echo $name_1
#echo $name_1_1

# 使用索引位置从右侧开始提取,从右边开始计算倒数第八个字符开始计数，取八个字符
# 如果是把字符后面的都取了，第三个参数也可以不用给，{i:0-8}
name_2=${i:0-8:8}
# 或者下面的也行，截取 右边的字符串
name_2_2=${i#*sra}
#echo $name_2
#echo $name_2_2


# 字符串的连接
full_name="$name_1$name_2"
# echo $full_name

mv $i $full_name

done


```







