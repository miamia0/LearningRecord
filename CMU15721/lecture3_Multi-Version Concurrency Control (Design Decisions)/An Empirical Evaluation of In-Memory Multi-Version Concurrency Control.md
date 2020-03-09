An Empirical Evaluation of In-Memory Multi-Version Concurrency Control

#####摘要
多版本并发控制(MVCC)是目前最火的事务管理算法。这篇文章进行了MVCC多核扩展上的讨论。 
并发控制协议 （concurrency control protocol） 版本存储（version storage） 垃圾管理（ garbage collection） 索引管理（index management）

#####2.2 DBMS Meta-Data
事务：DBMS给每个事务一个独立递增的时间戳。
元组：每一个物理version的meta-date都有四个字段：txn-id  begin-ts end-ts pointer
txn-id  每一个没有加写锁的元组这个字段都会被设置成0，大多数DBMS使用64位的txn-ied，所以就可以使用[compare and-swap (CaS)](https://blog.csdn.net/yanluandai1985/article/details/82686486) 指令来更新数值
如果id为$T_id$事务T想要更新元组A，DBMS检查A的txn-id是否是0，如果是0，那么DBMS将将设置txn-id为$T_id$。
begin-ts and end-ts 体现元组version的寿命，初始是0，结束时begin-ts设置为INF。
pointer指向之前或者之后的version。（*在什么情况下是之前什么情况下是之后？*）
####3. CONCURRENCY CONTROL PROTOCOL
协议的作用
(1) 判断是否允许事务访问或者修改一个指定版本的元组。 (2)是否允许事务提交修改。
MVCC对(phantom)幻读没有好处。
现有的提供可序列化事务的方法通常有 (1)index的锁 (2)事务提交时候的验证。
#####3.1Timestamp Ordering(MVTO)
在元组版本中新增加一个read-ts代表最后一个读取该元组的ts。DBMS在会在事务尝试读取或者更新写锁被其他事务持有的version时abort掉这个事务。

当事务T使用read操作读取元组A，DBMS查找一个version使$T_id在$begin-ts和end-ts字段范围之间，且tx-id==0或者==tid。

read需要检查当前事务的Tid是否落在begin-ts和end-ts之间，并且该tuple没有被别的事务锁定，即txn-id=0或等于Tid。如果可以读的话，将read-ts设置为tid。

*begin-ts 和end-ts代表当前version的有效区间，即在对于Tid为begin-ts 与end-ts之间的事务，这条version是合法的。*
当事务T需要加写锁，满足下面两个调教1.Tid>$B_x$->read-ts 2.$B_x$->tx-id==0代表该元组出现了新的修改，当前版本已经过期，所以创建新的版本$B_x+1$。在T结束后，设定$B_x+1$->begin-ts = Tid ,$B_x+1$->end-ts = INF,B_x->end-ts = Tid。代表Bx已经是旧版本$B_x+1$是当前版本。
*思考，如果在$T$运行过程中，出现了一个读操作$T_1$且$T_{1id}>T_id$，是否会读取$B_x$版本的数据？*

#####3.2Optimistic Concurrency Control(MVOCC)

分为三个阶段 read phase  validation phase write phase。
**read phase** 和MVTO基本相同。
**validation phase ** DBMS给事务另一个时间戳，然后判定read set是否已经被已经提交的事务更新了。如果没有冲突就进入 ** write phase** 新建一个version。并且begin-ts设定为T_commit,end-ts = INF。

#####3.3Two-phase Locking(MV2PL)
在这个方法里，读操作和写操作都需要获得锁。
write-lock使用txn-id，read lock 在meta-date里新增了一个read-cnt。
在对元组A进行读操作时，和其他方法一样DBMS首先寻找合法的元组版本。

####4. VERSION STORAGE
####5. GARBAGE COLLECTION
####6. INDEX MANAGEMENT
