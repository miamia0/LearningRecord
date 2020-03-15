#### 摘要
这篇论文使用了advance SIMD operation 包括gathers and scatters。
selections, hash tables, and partitioning 三种方案进行sort 与join的优化。

#####  三种并行方式
thread parallelism 数据分线程处理
instruction level parallelism 
data parallelism  使用SIMD

#### 3. fundimental operation
##### selective load

![load](pic\load.png)

##### selective store

![store](pic\store.png)

##### Scatter operation

![Scatter](pic\Scatter.png)

##### Gather operation

![gather](pic\gather.png)因为(L1) cache在1个时钟周期里只允许允许1个或2个不同位置的访问，gather 和 scatter 并不能在1个时钟周期内完成。

#### 4. SELECTION SCANS

![scan](pic\scan.png)

#### 5.HASH TABLES

利用SIMD的vector comparsion，使用多个hash table key与probe作比较， 

##### 5.1 Linear Probing

![5.1](pic\5.1.png)

##### 5.2 Double Hashing

##### 5.3Cuckoo Hashing

#### 6.BLOOM FILTERS

#### 7.PARTITIONING

#### 8. SORTING‘

#### 9.HASH JOIN

Xeon Phi

MIC 
