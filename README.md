<!--
 * @Descripttion: 
 * @version: 
 * @Author: cm.d
 * @Date: 2022-03-13 21:17:41
 * @LastEditors: cm.d
 * @LastEditTime: 2022-03-13 21:33:25
-->
# fast2022-paper-summary

个人的Fast 2022的paper的一些summary，全部论文参见：[fast22_full_proceedings.pdf](https://www.usenix.org/system/files/fast22_full_proceedings.pdf)

# Persistent Memory: Making it Stick

这个topic都是关于PM方向的文章，目前随着存储成本越来越低，PM在各大公司开始纷纷落地使用场景，FAST也专门开了一个针对PM方向的paper的topic

## NyxCache: Flexible and Efficient Multi-tenant Persistent Memory Caching

NyxCache，Wisconsin–Madison大学和Microsoft联合发的一篇paper，虽然标题是“一个灵活的、高效的多租户持久化内存”，但是其实看了下内容，这篇文章是个工程层面的文章，并不是PM的设计，而是一套基于PM的多租户framework，它本身不关注PM，而是在PM之上的多租户管控架构。文章列举了一些基于PM的多租户场景，比如基于PM的Memcache，由于PM存在严重的读写比性能差异(文章举了傲腾的例子)，导致在一个PM节点上的多个实例互相受到影响，所以基于这些问题，设计了NyxCache去管理PM集群，基于PM对于资源数据进行上报进行分析和管控，实现多租户隔离、共享资源分配、QoS等。

## HTMFS: Strong Consistency Comes for Free with Hardware Transactional Memory in Persistent Memory File Systems

HTMFS，上海交大的paper，偏理论性的论文，是基于HTM+PM设计的一个强一致、高性能的用户空间文件系统，注意这里并不是分布式的文件系统，这里的强一致仅仅是对localfs而言。阅读之前可能需要先对HTM有一些概念，建议先了解一下[事务内存](https://en.wikipedia.org/wiki/Transactional_memory)，不过文章设计的概念HOP更像是HyTM(文章自己也说了是hardware-software cooperative mechanism)，在HTM的缓存限制下，基于OCC做了并发控制，基于cooperative locks做了fallback，简单理解，就是在文件系统的读写操作接口中，通过HOP机制实现了操作的强一致性。

## ctFS: Replacing File Indexing with Hardware Memory Translation through Contiguous File Allocation for Persistent Memory

ctFS，Toronto大学的paper，简单点来理解，就是为了解决PM上文件查找效率的优化。原本的PM的访问要基于MMU进行控制(和RAM一样，熟悉操作系统内存管理很容易理解)，因此主要性能消耗在了块的地址查找上，文中举了例子ext4-DAX，即便用了DAX，也有45%的性能浪费在了对文件偏移量转换到PM的实际地址上，因此设计了ctFS文件系统，它的思路是将文件映射到虚拟内存的连续空间中，那么对于文件偏移到PM地址的查找，就变成了简单的映射关系，只需要维护一个小容量的索引。文章还通过使用该fs对leveldb的读写进行了优化，性能比基于ext4-DAX高出了3.6倍。

## FORD: Fast One-sided RDMA-based Distributed Transactions for Disaggregated Persistent Memory

TODO

# A Series of Merges

## Closing the B+-tree vs. LSM-tree Write Amplification Gap on Modern Storage Hardware with Built-in Transparent Compression

一二作是Rensselaer和Google，文章比较偏工程化，是通过现代硬件对于b+树进行优化。目前的存储领域里，由于LSM-tree的高写入、低空间优势，很多DB的存储引擎都使用LSM-tree去替换b+ tree，b+ tree通常在随机读的场景更有优势。这篇文章中，作者对b+ tree利用具有透明无损压缩能力的硬件设备来进行写放大和存储空间的优化来论证在某些场景中可以替换lsm-tree。基于该硬件优化后的b+树能将原来的写入放大降低10倍，并在rocksdb中取得了比lsm-tree更好的写放大效果。

## TVStore: Automatically Bounding Time Series Storage via Time-Varying Compression

TVStore，清华大学+华为发的paper，并且基于此概念开发了一个开源项目[Apache IoTDB](https://iotdb.apache.org/)，不过这个开源项目工程味比较浓，整合了大量的针对工业互联网的数据采集和分析工具，paper本身还是关注了时序数据的存储、压缩算法。paper引入了一种叫做Time-Varying的压缩算法，其实就是定义了基于时间维度的压缩函数，文章认为用户数据的压缩场景，比如有损或者无损、压缩比率与数据的重要度有关，而数据的重要度与时间函数有关，因此，最终算法被设定为一个类似基于时间窗口的分段压缩算法(支持流与批处理)，并实现了一个框架TVC，除了支持通过时间函数进行压缩，还支持指定阈值(存储空间)的自动压缩。非常值得一提的是，我翻了下Apache IoTDB的文档，里面暂时不建议开启时间分区，2333333

## Removing Double-Logging with Passive Data Persistence in LSM-tree based Relational Databases

山东大学为主力的paper，厉害了。文章要解决的问题是在基于LSM-Tree的关系数据库存储中，需要两次写入WAL的问题，一次是RDB自己的WAL，一次是LSM-Tree的WAL。文章提出了一个新模型PASV，去掉了RDB自身的WAL，基于被动存储性原理实现了RDB层面的容灾方案。

# Solidifying the State of SSDs

SSD方向的paper，比较硬核，几篇文章都是涉及到硬件层面的优化，主要focus在ssd的寿命上

## Improving the Reliability of Next Generation SSDs using WOM-v Codes

今年的best paper！，google和Toronto一起发表的paper，paper围绕的是ssd的读写寿命的优化。早期的ssd都是基于SLC，每个cell只存储一个bit，随着对存储容量的需求，现在的闪存通常会在cell中存储多个bit，而经过作者的测试表明，cell中bit的数量与其能支持的擦除次数成反比关系，每多存一个bit，就会导致擦除次数少一个数量级。paper中为了解决这个问题，设计了新的ssd的存储编码non-binary, Voltage-Based Write-Once-Memory (WOM-v) Codes，使用很低的性能损耗实现空间的优化，并且设计了基于FEMU的模拟器进行测试。

## GuardedErase: Extending SSD Lifetimes by Protecting Weak Wordlines

GuardedErase，韩国几个大学出的paper，要解决的问题也是ssd的寿命问题，是针对3d Nand存储的优化方案，有点硬核。文章对于Nand flash中同一闪存块中不通的wordlines的可靠性不同，通过新的硬件设计来降低弱wordlines上的擦除电压，来尽量平衡各wordlines的可靠性，实现Nand flash寿命的提升。感兴趣的可以深入看看。

## Hardware/Software Co-Programmable Framework for Computational SSDs to Accelerate Deep Learning Service on Large-Scale Graphs

机器学习方向的SSD探索，TODO

## Operational Characteristics of SSDs in Enterprise Storage Systems: A Large-Scale Field Study

Toronto大学的paper(今年真的猛)，基于某个SSD厂商向各大企业提供的将近200w的SSD盘进行大规模的数据分析得到的一些数据报告，值得深入读一下。

# Distant Memories of Efficient Transactions

# The Five Ws of Deduplication

# Meet the 2022 File System Model-Year Lineup

# Keys to the Graph Kingdom

# Keeping the Fast in FAST

