# Hadoop 之 文件切分算法

文件切分算法主要用于确定 InputSplit 的个数，以及每个 InputSplit 对应的数据段。FileInputFormat 以文件为单位切分生成 InputSplit。
对于新旧 MapReduce 有各自确定 InputSplit 大小的计算公式。
在旧方法中，由以下三个属性值确定其对应的 InputSplit 的个数： 
1. goalSize：根据用户期望的 InputSplit 数目计算出来的，即 totalSize/numSplit。 其中，totalSize 为文件的总大小；numSplit 为用户设定的 Map Task 个数，默认为1； 
2. minSize：InputSplit 的最小值，由配置参数 mapred.min.split.size 确定，默认为1； 
3. blockSize：文件在 HDFS 中存储的 block 大小，默认为64MB。
这三个参数一起决定 InputSplit 的最终大小，方法如下：
```splitSize = max{minSize, min{goalSize, blockSize}}```
在新方法中，InputSplit 的划分不再考虑用户指定的 Map Task 个数，用 mapred.max.split.size 替代，记为 maxSize。计算公式如下：
```splitSize = max{minSize, min{maxSize, blockSize}}```




### ###  **特别注意 **

默认情况下，split 大小和 block 大小是相同的。
举例:文件 S 被分成 3 个 block 块，分别位于 A，B，C 三个节点，如果 InputSplit 的大小大于 block 的大小，则对于一个输入分片，他要去其他节点取数据后再组成 InputSplit。这样会产生网络传输，降低Map Task的本地性，降低效率。
所以，最好使 InputSplit 大小与 block 大小相同。



