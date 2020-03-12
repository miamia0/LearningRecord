#### 什么是scheduling

对于每一个查询，DBMS必须确认where when how来执行这个查询。

* 需要使用多少task？（how）
* 需要使用多少CPU资源？(how)
* task在哪个CPU上执行？(where)
* task应该在哪里存储他的输出？(where)

#### 进程调度模型(Process Models)
worker的定义:DBMS组件，负责代表客户机执行任务并返回结果

##### 1 一worker一进程 PROCESS PER WORKER
每一个worker都是一个OS进程。
使用OS的调度方法。
为全局数据结构使用共享内存。
进程崩溃不会导致整个系统崩溃。
Examples: IBM DB2, Postgres, Oracle。
（老师这里提到DB2有四个完全不一样的版本，每个版本的代码都完全不一样，所以可能会有同样是DB2，但是采取了完全不同的管理方式）
##### 2 进程池 Process Pool

work使用进程池中空闲的进程
不利于CPU的cache。

Examples: IBM DB2 Postgres(2015)
##### 3 一worker一线程 Thread per DBMS Worker
DBMS管理自己的scheduling。可能会使用调度线程。
线程崩溃可能会导致整个系统崩溃。
优点：1.更少的上下文切换花销。2.不需要管理共享内存。
老师说自己不知道近十年来还有什么数据库是不用线程的，除非是fork from Postgres。以前使用进程主要是因为早期的操作系统没有提供线程的接口。

scheduler需要知道系统的硬件的布局。

#### 数据处理 Data Placement
DBMS可以将存储器分成很多份，然后把各个部分分配给CPU。在调度的时候可以吧worker给最近的CPU。
##### 内存申请
malloc 操作在申请内存时不一定代表内存有剩余如此足够的大小，使用虚拟内存，在正式访问的时候产生页错误才会申请物理内存。

#### 在NUMA系统中的内存分配方式
方式 #1: 交叉存取 Interleaving
在cpu之间均匀分配分配的内存。
方式 #2: First-Touch
在访问导致页面错误的内存位置的CPU上。（不是申请内存的线程）

#### partitioning & placement
partitioning 
分区方案用于根据某些策略分割数据库。
比如 循环，设定范围，hash，

placement
→循环,跨核
placement的方案需要知道partitioning的方案，这样可以提高效率。
#### Scheduling

MORSEL-DRIVEN SCHEDULING
NUMA-aware operator
SAP HANA NUMA-AWARE SCHEDULER

