---
layout: post
author: Kuang
title: 现代硬件架构下的并行Hash Join算法2
categories: Database
tags: join
---

论文Cagri Balkesen, Jens Teubner, Gustavo Alonso, M. Tamer Özsu: Main-memory hash joins on multi-core CPUs: Tuning to the underlying hardware. ICDE 2013: 362-373




### 1. 前言

影响哈希连接算法的因素主要有两点：一是cache命中率，因为CPU在cache中读取数据的速度远高于在内存中读取数据的速度，所以希望在处理数据时可以尽量可以在cache中找到想要的数据，而不希望过多地访问内存；二是多线程带来的同步代价，线程同步需要使用锁机制来保证数据的一致性，而使用锁会带来较大的开销。

当前主流的哈希连接算法主要有两类：一类算法对硬件参数不敏感，即无论在什么硬件环境下，算法的参数都是一样，无需对参数进行调整。这类算法的代表就是在本章另一篇文档[Parallel Hash Join #1][1]中提到的不分区的哈希连接算法。这类算法的支持者认为，现代的硬件已经能够很好地隐藏cache丢失带来的影响，而为了减少cache丢失而使用的分区操作会增加线程同步的代价，因此不关心硬件条件的不分区哈希连接算法，比对硬件参数敏感的有分区哈希连接算法更高效。而另一些研究者对此持相反观点，他们认为当前硬件仍然需要仔细考虑cache命中率对算法性能的影响，对硬件参数敏感的有分区哈希连接算法更加高效。

针对以上争议，本文通过实验的方式比较不同条件下各种哈希算法的性能，讨论不关心硬件参数(Hardware-Oblivious)的哈希连接算法，和对硬件参数敏感(Hardware-Conscious)的哈希连接算法的优劣。

### 2. 哈希连接算法

#### 2.1 Hardware-oblivious hash join

Hardware-oblivious hash join顾名思义就是其不需要根据硬件环境的变化来调整算法参数。在本章另一篇文档[Parallel Hash Join #1][1]中提到的no partitioning hash join就属于这一类算法，不分区的哈希连接算法分为两个阶段：build和probe。在build阶段，所有线程同步构建一张哈希表。在probe阶段，所有线程共同工作，在哈希表中找到与S表中元组相匹配的桶。不分区的哈希连接算法如图2.1所示。

　　　　　　　　　　　![][2]

　　　　　　　　　　　　　　图2.1 No partitioning hash join

#### 2.2 Hardware-conscious hash join

与hardware-oblivious hash join相反，hardware-conscious hash join对硬件环境较为敏感，在不同参数下的算法性能可能差别较大，因此往往需要根据硬件参数仔细调整算法参数。文档[Parallel Hash Join #1][1]中提到的partitioned hash join算法就属于这一类算法。有分区的哈希连接算法过程分为3个阶段: partition、build和probe阶段。partition阶段将R表的数据划分到不同的分区上，build阶段在各个分区上构建哈希表，probe阶段先将S表中的元组映射到对应的分区上，再进行连接操作，如图2.2所示。

　　![][3]

　　　　　　　　　　　　　　图2.1 Partitioned hash join

### 3. 实验

为了比较两类算法的性能，首先测试hardware-oblivious hash join在不同硬件参数下的表现，然后测试hardware-conscious hash join在不同硬件参数下的表现，最后对比两类算法的优劣。

#### 3.1 实验数据集和实验环境

为了与Blanas等人\[1\]以及Kim\[2\]等人的实验对比，这里分别使用与他们一样的数据集，如表3.1所示。

　　　　![][4]
　　　　　
　　　　　表3.1 实验数据集（A是文献\[２\]使用的数据集，B是文献\[１\]使用的数据集）

硬件架构有４种，分别是Intel Nehalem、Intel Sandy Bridge、AMD Bulldozer和Sun Niagara 2。各种硬件的具体参数如表3.2所示。

　　![][5]

　　　　　　　　　　　　　　　表3.2 实验使用的硬件架构详细参数

#### 3.2 Hardware-oblivious hash join

No partitioning join算法不需要根据硬件环境调整参数，是Hardware-oblivious算法的一种。分析no partitioning join算法的代价，如下式所示。

　　　　　　　　　　![][6]

如上述公式所示，对于不分区的哈希连接算法，整体的代价应该是build阶段的代价加上probe阶段的代价。其中Cput是向哈希表中写入一项所需的代价，Cget是从哈希表中读取一向所需的代价，由于写数据时存在锁竞争的问题，因此Cput会略大于Cget。因此使用数据集A测试不分区的哈希连接算法，则build阶段的代价应该至少占整个阶段的1/16，但是根据文献\[2\]所述，build阶段的消耗只占不到2%，这非常值得质疑。

因此研究Blanas等人\[2\]等人的代码，发现他们在做实验时，build阶段的数据是预先经过排序的。如果将这个预先排序操作去掉，重新进行试验，发现build阶段耗费的时间增加了4倍，而且数据未预先进行排序时，cache丢失明显增加。这就证明，预先排序对于cache命中率提升有所帮助。但是预先排序所需的代价Blanas等人并未计入到算法的代价中。这就是为什么在Blanas等人的工作中build阶段花费的代价不到整个过程的2%。

为了减少build阶段的cache丢失，可以考虑优化哈希表的结构。Blanas等人设计的哈希表结构如图3.1所示。向这种结构的哈希表中插入一个元组需要先从锁列表中获取写同步锁，再从哈希表中获得头指针，最后寻找可以插入的位置。这3步操作的每一步都有可能造成cache丢失。因此作者设计了另一种哈希表。这种哈希表的头部直接放在桶中，8个字节的头部包含了1字节的同步锁以及当前桶中元组的数量。这种设计有效地减少了cache丢失的问题。

　　　　　![][7]

　　　　　　　　　　　　　　　　图3.1 传统的哈希表结构示意图

　　　　　　　　　　![][8]

　　　　　　　　　　　　　　　　　　图3.2 优化的哈希表结构

#### 3.3 Hardware-conscious hash join

Radix hash join在partition阶段对数据进行多趟分区，每趟分区操作限制分区的数量,如图3.3所示。
　![][9]

　　　　　　　　　　　　　　　图3.3 Radix hash join 示意图

Radix hash join的性能受其分区数影响，不同硬件条件下的最佳分区数不尽相同(如图3.4所示)，因此Radix hash join是一种hardware-conscious的哈希连接算法。

![][10]

　　　　　　　　　　　图3.4 不同分区数对radix hash join性能的影响

分区的哈希连接算法缓解了cache丢失问题，但是带来了更多的线程同步代价。线程各自对一部分数据进行操作，因此线程之间存在负载均衡的问题。radix hash join可以大致分为5个阶段：计算R表的直方图;计算S表的直方图;第一趟分区操作;第二趟分区操作;连接操作。图3.5所示的是未考虑负载均衡算法各阶段所需的时钟周期。图3.6所示的是解决负载均衡问题后算法各阶段所需的时钟周期。可以看到，不考虑负载均衡时，线程等待时间较长，因此浪费了很多资源，总的运行时间较长。

　　　　　　　　　![][13]

图3.5 不考虑负载均衡时各线程运行情况(使用数据集A,其中纯色区域表示线程运行时间，阴影区域表示线程等待时间)

　　　　　　　　　　![][14]

　　　　　　　　　　　　　图3.6 考虑负载均衡问题时各线程运行情况

#### 3.4 Hardware-oblivious join与Hardware-conscious join的比较

在不同硬件环境下，no partitioning哈希连接算法和radix哈希连接算法的表现如图3.7所示。可以看到，在硬件环境一样的情况下，radix哈希连接几乎总是优于no partitioning哈希连接。同时可以发现在数据集A上，两种算法的差异不如使用数据集B时的差异。因此可以认为，当R表和S表元组数量差距较大时对不分区的算法有利。

![][11]

图3.7 各种硬件架构下两种算法运行所需的时钟周期(Nehalem为6线程，Sandy Bridge和AMD为16线程，Niagara为64线程)

算法的可扩展性也是评价其优劣的一个指标，好的算法具有较强的可扩展性。no partitioning哈希连接算法和radix哈希连接算法的可扩展性实验结果如图3.8所示。可以看出，两种算法的可扩展性都较好，radix哈希连接算法可扩展性更强。

![][12]

　　　　　　　　　　　　图3.8 不同硬件架构下两种算法的性能与线程数的关系

数据集分布对算法性能有较大的影响，测试hardware-oblivious连接算法和hardware-conscious连接算法在不同数据分布下的表现，实验结果如图3.9所示。

![][15]

图3.9 不同数据分布下两种算法的表现(Zipf factor z指的是Zipf分布的参数，z越大表示数据倾斜程度越高)

从图3.9中可以看出，no partitioning连接算法的确可以从分布不均匀的数据中受益，但是其性能仍然不如radix连接算法。并且radix哈希连接算法在不同分布的数据集下表现较为稳定。

如图3.7所示，no partitioning连接算法和radix哈希连接算法的差异在数据集A和数据集B上不同。观察数据集A和数据集B发现两者的主要差异是R表与S表大小的比例。图3.10所示的是在不同比例下两种算法的表现。可以看出，no partitioning连接算法受该参数影响较大，而radix连接算法表现较为稳定。

　![][16]

　　　　　　　　　　　　图3.10 R表与S表大小比例对算法性能的影响


### 4 总结

通过实验可以看出，现代的硬件架构并不能很好地隐藏cache丢失带来的影响，hardware-oblivious连接算法性能通常不如使用适合参数的hardware-conscious连接算法。另外，在某些情况下，hardware-conscious算法具有更好的稳定性。因此在设计哈希连接算法时应当仔细考虑硬件环境，调整算法的参数。


### 参考资料

\[1\] C. Kim, E. Sedlar, J. Chhugani, T. Kaldewey, A. D. Nguyen, A. D. Blas,
V. W. Lee, N. Satish, and P. Dubey, “Sort vs. hash revisited: Fast join
implementation on modern multi-core CPUs,” PVLDB, vol. 2, no. 2, pp.
1378–1389, 2009.

\[2\] Spyros Blanas, Yinan Li, Jignesh M. Patel: Design and evaluation of main memory hash join algorithms for multi-core CPUs. SIGMOD Conference 2011: 37-48

[1]: https://gitee.com/daseATecnu/bds2017/wikis/Parallel-Hash-Join-%231?parent=%E5%B9%B6%E8%A1%8C%E8%BF%9E%E6%8E%A5
[2]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/nopartitionfigure.PNG
[3]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/partitionfigure.PNG
[4]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/docworkload.PNG
[5]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/hardwarepara.PNG
[6]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/mathequation.PNG
[7]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/blanashash.PNG
[8]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/kihash.PNG
[9]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/radixhash.PNG
[10]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/radixbitperformance.PNG
[11]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/performancecompare2.PNG
[12]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/scability.PNG
[13]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/nobalance.PNG
[14]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/havebalance.PNG
[15]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/distributioninfluence.PNG
[16]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/ratioinfluence.PNG