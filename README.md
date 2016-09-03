# itbook
1. 统计线程内存使用情况

/proc/pid/statm
包含了所有CPU活跃的信息，该文件中的所有值都是从系统启动开始累计到当前时刻。

输出解释
CPU 以及CPU0。。。的每行的每个参数意思（以第一行为例）为：
参数 解释 /proc/ /status
    Size (pages) 任务虚拟地址空间的大小 VmSize/4
    Resident(pages) 应用程序正在使用的物理内存的大小 VmRSS/4
    Shared(pages) 共享页数 0
    Trs(pages) 程序所拥有的可执行虚拟内存的大小 VmExe/4
    Lrs(pages) 被映像到任务的虚拟内存空间的库的大小 VmLib/4
    Drs(pages) 程序数据段和用户态的栈的大小 （VmData+ VmStk ）4
    dt(pages) 0 

例如统计namenode占用内存大小: cat /proc/namenode_pid/statm | awk '{print $2*4/1024"M"}'
---
2. NameNode内存解析及大小评估
http://bigdatadecode.club/NameNode%E5%86%85%E5%AD%98%E8%A7%A3%E6%9E%90%E5%8F%8A%E5%A4%A7%E5%B0%8F%E8%AF%84%E4%BC%B0.html
