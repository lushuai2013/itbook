# Hadoop中map数的计算
Hadoop中在计算一个JOB需要的map数之前首先要计算分片的大小。计算分片大小的公式是：

goalSize = totalSize / mapred.map.tasks

minSize = max {mapred.min.split.size, minSplitSize}

splitSize = max (minSize, min(goalSize, dfs.block.size))

totalSize是一个JOB的所有map总的输入大小，即Map input bytes。参数mapred.map.tasks的默认值是2，我们可以更改这个参数的值。计算好了goalSize之后还要确定上限和下限。

下限是max {mapred.min.split.size, minSplitSize} 。参数mapred.min.split.size的默认值为1个字节，minSplitSize随着File Format的不同而不同。


上限是dfs.block.size，它的默认值是64兆。

举几个例子，例如Map input bytes是100兆，mapred.map.tasks默认值为2，那么分片大小就是50兆；如果我们把mapred.map.tasks改成1，那分片大小就变成了64兆。

计算好了分片大小之后接下来计算map数。Map数的计算是以文件为单位的，针对每一个文件做一个循环：

1.文件大小/splitsize>1.1，创建一个split，这个split的大小=splitsize，文件剩余大小=文件大小-splitsize

2.文件剩余大小/splitsize<1.1，剩余的部分作为一个split

举几个例子：

1.input只有一个文件，大小为100M,splitsize=blocksize,则map数为2，第一个map处理的分片为64M,第二个为36M

2.input只有一个文件，大小为65M,splitsize=blocksize，则map数为1，处理的分片大小为65M （因为65/64<1.1）

3.input只有一个文件，大小为129M,splitsize=blocksize，则map数为2，第一个map处理的分片为64M,第二个为65M

4.input有两个文件，大小为100M和20M,splitsize=blocksize,则map数为3，第一个文件分为两个map，第一个map处理的分片为64M,第二个为36M，第二个文件分为一个map，处理的分片大小为20M

5.input有10个文件，每个大小10M，splitsize=blocksize，则map数为10，每个map处理的分片大小为10M

再看2个更特殊的例子：

1.输入文件有2个，分别为40M和20M，dfs.block.size = 64M， mapred.map.tasks采用默认值2。那么splitSize = 30M ，map数实际为3，第一个文件分为2个map，第一个map处理的分片大小为30M，第二个map为10M；第二个文件分为1个map，大小为20M

2.输入文件有2个，分别为40M和20M，dfs.block.size = 64M， mapred.map.tasks手工设置为1。

那么splitSize = 60M ，map数实际为2，第一个文件分为1个map，处理的分片大小为40M；第二个文件分为1个map，大小为20M

通过这2个特殊的例子可以看到mapred.map.tasks并不是设置的越大，JOB执行的效率就越高。同时，Hadoop在处理小文件时效率也会变差。

根据分片与map数的计算方法可以得出结论，一个map处理的分片最大不超过dfs.block.size * 1.1 ，默认情况下是70.4兆。但是有2个特例：

1.Hive中合并小文件的map only JOB，此JOB只会有一个或很少的几个map。

2.输入文件格式为压缩的Text File，因为压缩的文本格式不知道如何拆分，所以也只能用一个map。
