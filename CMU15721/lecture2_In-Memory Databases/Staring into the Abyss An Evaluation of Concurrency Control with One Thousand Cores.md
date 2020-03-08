#### 摘要
现在计算机芯片上的核心数越来越多，并发控制问题也越来越突出。
这篇论文在主存DBMS，设定系统核心数是1024个的cpu（模拟方式），对7个并发控制算法进行实验。
实验结果是所有的算法在这样的配置下失败了，在多核结构下，数据库系统应该重新制作体系结构。

#### 1.介绍
单核速度提升的时代已经过去，现在的提速渐渐的开始靠多核来进行cpu提速。多线程并行运行的主要问题是数据的并发访问。所以并发控制显得尤为重要。这是第一篇针对在大规模多核cpu下运行主存DBMS算法评估的论文。
这篇论文的三点主要贡献
* 一个对于7个并发控制算法的可扩展性(scalability)综合评估。
* 第一个在1000+核上基于OLTP DBMS 进行评估的论文。
* 提出了并发控制算法的瓶颈
这篇论文第二章使用评估结果对并发控制算法进行介绍，第三章介绍了系统的各个部分，第四章和第五章展示了分析结果，第六章进行结果的讨论，第七章介绍了相关工作，第八章讨论了未来的研究方向。
#### 2.并发控制算法
这里将分成并发控制算法两类
* two-phase locking  [二段锁协议](https://www.cnblogs.com/feng9exe/p/10697838.html)
* timestamp ordering protocols  时间戳排序协议

|      |           |                                            |
| ---- | --------- | ------------------------------------------ |
| 2PL  | DL_DETECT | 2PL with deadlock detection.               |
| 2PL  | NO_WAIT   | 2PL with non-waiting deadlock prevention.  |
| 2PL  | WAIT_DIE  | 2PL with wait-and-die deadlock prevention. |
| T/O  | TIMESTAMP | Basic T/O algorithm.                       |
|T/O   |MVCC       | Multi-version T/O.							|
|T/O   |OCC        |Optimistic concurrency control.				|
|T/O   |H-STORE     | T/O with partition-level locking.			|
##### 2.1 二段锁协议(Two-Phase Locking)
2PL是第一个可以被证明的正确的方法，在此方案下，事务必须得到数据库中的特定元素获取锁，然后才能对该元素执行读或写操作。比如事务必须在读之前获取**读锁**(read lock)，在写之前获取**写锁**(write lock)。
这个协议有两条规定 

 1. 不同的事务不能同时拥有**冲突锁**（conflicting locks）
 2. 如果一个事务开始释放锁后，就不能获得新的锁。

**在同一元素上读锁与写锁冲突，写锁与写锁冲突。**
2PL中的两个阶段介绍: 第一阶段，允许事务获得需要的任意数量的锁，不能释放锁。当事务释放锁时，它进入第二个阶段，此时禁止获取其他锁。当事务终止时(通过提交或异常中止)，所有剩余的锁将自动释放。

2PL是悲观算法。如果一个事务不能为一个元素获取锁，那么它将被迫等待，直到锁变得可用。那么DBMS就会有产生死锁的可能性。因此，**2PL的各种变体之间的主要区别在于它们如何处理死锁，以及在检测到死锁时所采取的操作**。

###### 2PL with Deadlock Detection (DL_DETECT)带死锁检测的2PL:
 - DBMS 维护一个事务等待图并且检查是否有环(有环则有产生死锁的可能) 如果有环则强行中止一个被选中的事务然后进行重启。检测器根据事务已经使用的资源数量来选择中止哪个事务。
###### 2PL with Non-waiting Deadlock Prevention (NO_WAIT)带死锁预防的2PL:
 - 当系统怀疑死锁可能发生时，拒绝加锁请求，在事务的加锁请求被拒绝的信息后，事务立即中止。
###### 2PL withWaiting Deadlock Prevention (WAIT_DIE)带等待死锁预防的2PL:
 - 这是NO_WAIT方案技术的一种非抢占式(non-preemptive)变体，允许事务等待持有所需锁的事务(如果该事务比持有锁的事务更早)。如果请求的事务时间戳较小，则它将被中止并重新启动。每个事务在执行之前都需要获得一个时间戳，而时间戳排序保证不会发生死锁。
##### 2.2 时间戳排序协议
两种方案之间的关键区别在于(1)DBMS检查冲突的粒度(即数据库管理系统的粒度 元组与划分)和(2) DBMS检查这些冲突的时间(事务正在运行或在结束时)。
###### Basic T/O (TIMESTAMP):
  在读或修改时与随后一个读或写入事务比较时间戳，小于则拒绝。在写入时，如果小于最后一个读时间戳，就拒绝。读创建元组的本地副本，以确保可重复读取，因为它不受锁的保护。
###### Multi-version Concurrency Control (MVCC):
写操作创建新副本，读操作不阻塞。
###### Optimistic Concurrency Control (OCC): 
写操作在私有空间，判定写操作与其他读操作是否有交集。在本文里采用类似于 Microsoft’s Hekaton 的实现。
###### T/O with Partition-level Locking (H-STORE):
 数据库分成不相交的部分，每一个部分都有一个锁和一个独占线程，事务需要获得它访问的所有部分的锁。
#### 3. MANYCORE DBMS TESTBED
#### 4. 设计选择&优化(DESIGN CHOICES & OPTIMIZATIONS)
##### 4.1 General Optimizations
###### 内存申请 memory allocation
 - 原始的linux的malloc函数 速度很慢
 - 优化后的malloc:[TCMalloc](https://wallenwang.com/2018/11/tcmalloc/)(Thread-Caching Malloc) [jemalloc](http://www.cnhalo.net/2016/06/13/memory-optimize/) 效果也不好
 - 重写的malloc: 类似于TCMalloc 和jemalloc，每一个线程都有自己的内存池，不同的是，这里的内存池大小会根据工作负载动态的调整。
###### Lock Table:
没有使用集中的锁表或时间戳管理器，而是以每个元组的方式实现这些数据结构，其中每个事务只锁住它需要的元组。
###### Mutexes
 2PL ， 死锁检测器的互斥锁是主要瓶颈。T/O，用来时间戳的申请互斥锁是主要瓶颈。

####4.2 Scalable TwoPhase Locking
##### Deadlock Detection 

DL_DETECT 中死锁检测是瓶颈，许多线程竞争wait-for图。本文的实现是按照核进行数据结构的划分，从而不需要对wait-for 加锁。当一个事务要更新wait-for图时，它的线程使用它正在等待的事务更新它的队列，而不使用任何锁。

#####Lock Thrashing:
(lock thrashing)指一个事务直到提交前都会持有锁的问题。
使用YCSB workload进行测试(YCSB是工作负载的测试工具，可以设定读写比率)得到如下的图。

由图像得这是lockbased最大的瓶颈。
#####Waiting vs. Aborting:
增加timeout threshold

####4.3 Scalable Timestamp Ordering

#####Timestamp Allocation
###### atomic addition with batching
* DBMS 使用同样的原子指令申请时间戳，但是时间戳管理器同时返回多个时间戳。
###### CPU clocks
* 每个线程使用自己的逻辑时钟并且和线程id进行链接。在分布式系统中，使用软件协议或者内部时钟可以达到同步。在2014俩，只有intel 的cpu支持跨核同步时钟。
###### hardware counters
* 最后，第三种方法是使用高效的内置硬件计数器。计数器的物理位置位于中央处理器的中心，这样到每个核心的平均距离就最小化了。目前没有现有的CPU支持这个功能。因此，我们在Graphite中实现了一个计数器，其中时间戳请求通过芯片上的网络发送。

結果:
mutex-based 效果最差~1million
atomic addition 30million ts/s 当核数量增多时降低到8million，这是由于缓存一致性流量从写回和无效的最后一个副本对应的cacheline每一个时间戳。在核之间通信需要大约100个时钟周期。在1GHz下的吞吐量为10milionts/s。
hardware counter-based approach 1 billion ts/s，因为只需要一个时钟周期。


我们还在DBMS中测试了不同的分配方案，以了解它们在实际工作负载下的执行情况：
在写密集型的任务中fig 7a，如果线程间没有竞争，则和之前的fig6实验结果是一样的，但是如果线程间有许多竞争，情况就大不一样了。 
* batched atomic addition 效果更差，原因是事务发生冲突时重启，在同一线程上，但是分配的新时间戳。
*  non-batched atomic addition 和clock 与hardware方法一样好。
所以这篇文章使用non-batched atomic addition因为不需要特殊的硬件支持

#####Distributed Validation

通过单独元组验证来解决OCC读操作的最后临界阶段，将这个检查分解为更小的操作。

#####Local Partitions:
因为DBMS的所有线程都在同一个进程里，允许多部分的事务可以直接访问远程分部。这样使进程内部交流更快。所有线程在不进行复制的情况下访问只读表，从而减少了内存占用。

#### 5. 实验分析
: (1) scalability  (2) sensitivity 
##### 5.1只读任务 Read-Only Workload
* DL_DETECT 和NO_WAIT 都没有contention，所以不会 abort，效率高。
* WAIT_DIE 和 MVCC 因为 allocate timestamp为时间瓶颈
* OCC 和 TIMESTAMP 因为要复制元组保证重复读，所以性能低。

#####5.2  写密集任务Write-Intensive Workload
* DL_DETECT效率不好，因为wait for graph出现了大量的lock thrashing。NO_WAIT是最scalable的因为取消了等待。
* NO_WAIT WAIT_DIE 有很高的abort 概率，但是因为一般等待的时间比重启的时间要更长，所以效果仍然是比较好的。（但是现实里，可能多个table index等需要回滚，所以效果可能没有试验中那么好）
* MVCC 比较好，因为多版本能够为 时间戳较小的读操作提供服务。
* OCC 因为每个事务都必须在冲突解决之前完成，所以被abort的事务仍然需要花费时间。

在high contention下。
* NO_WAIT 持有锁时间较短。
* OCC 至少可以提交一个ts，所以性能要好一些
#####5.3 Working Set Size
* DL_DETECT and NO_WAIT的性能随着锁持有时间增长而下降
* T/O主要的时间耗费仍然是申请时间戳，fig8a 9b显示申请时间戳不再是主要的瓶颈。
* T/O算法比DL_DETECT对contention有更好的承受能力
#####5.4 Read/Write Mixture
* 每个算法都在读操作占比更多的时候表现比较好。
* TIMESTAMP 和 OCC 要表现不好因为需要读数据的时候进行复制
* MVCC 在 write request 较少时性能较高，因为不阻塞读操作
#####5.5 Database Partitioning
#####5.6 TPC-C
######5.6.1 4 Warehouses
######5.6.2 1024 Warehouses
H-STORE的总体性能最好，因为能够利用分区。
####6 DISCUSSION
|      |           |                                            |
| ---- | --------- | ------------------------------------------ |
| 2PL  | DL_DETECT | Scales under low-contention. Suffers from lockthrashing|
| 2PL  | NO_WAIT   | Has no centralized point of contention. Highly scalable. Very high abort rate. |
| 2PL  | WAIT_DIE  | Suffers from lock thrashing and timestamp bottleneck. |
| T/O  | TIMESTAMP | High overhead from copying data locally. Nonblocking writes. Suffers from timestamp bottleneck.|
|T/O   |MVCC       |Performs well w/ read-intensive workload.Nonblocking reads and writes. Suffers from timestamp bottleneck.|
|T/O   |OCC        |High overhead for copying data locally. High abort cost. Suffers from timestamp bottleneck.|
|T/O   |H-STORE     | High overhead for copying data locally. High abort cost. Suffers from timestamp bottleneck.|
####7. RELATED WORK

####8. FUTURE WORK
software-hardware co-design is the only solution to address these issues.
####9. ACKNOWLEDGEMENTS

####10. CONCLUSION
 结论是目前没有算法已经在超多核的情况下得到比较好的效率。
 2PL方法在短事务和低竞争的情况下表现优秀类似于key-value任务。T/O在多竞争和长事务情况下优秀类似于复杂的OLTP任务。我们提出了一些研究方向，我们计划进行探索以纠正这些扩展问题。

参考文献:
https://blog.csdn.net/rsy56640/article/details/91455178
