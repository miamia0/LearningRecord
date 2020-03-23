当前进度

- [x] Lab 1
- [x] Lab 2
   - [x] Lab 2A
    - [x] Lab 2B
    - [x] Lab 2C
- [x] Lab 3
   - [x] Lab 3A
   - [x] Lab 3B
- [ ] Lab 4

  - [ ] Lab 4A
  - [ ] Lab 4B
  

### Lab1

---



### Lab2

---



LAB2遇到的一些问题

* 因为Leader在接收到更大的term的时候需要转变回Follower，但是如果转变term后仍然发送了heartbeat会覆盖其他server的log。

* candidate，leader在接收到合法的requestvote或者heartbeat的时候应该立即转变为follower。

* follower在收到任意不合法的requestvote时仍然需要将自己的term进行更新。

* 因为需要用到log的长度作为election的依据，所以在更新log的时候必须将之后的无用log完全丢弃。

* 因为candidate的选举是使用随机时钟来保证每个follower都有被选举的机会，所以在测试条件不是很好的情况下，随机时钟可以将随机返回扩大一些。

* 状态的转变必须是立即的，首要进行的。因为状态代表着进行的操作：leader：发送heartbeat，candidate：进行election，follower：等RPC。在term转变之前的，term和status的相互作用是Raft协议的基础。

  * 一般term的转变都是从低term转变为高term，而高term代表着发言权，低term可以被任意高term所拒绝。
  * 高term的leader可以有至高无上的权力。
  * 低term的leader代表已经是out-of-date的leader
  * 高term的follower可以提示低term的leader转变为follower
  * 低term的candidate遇到高term的follower，leader，candidate都要转变为follower。
  * **leader只会因为变为follower才会转变term**，leader在接收到requestvote时是忽视的，candidate则会判定term。

  

* 多线程同步真的很重要。。

  lab2C需要把currentterm，votefor，log进行persist。

  currentterm和log进行presist比较好理解，而votefor进行persist可能是为了加速重启的时候进行leader选举而进行的举动。等于变相存储了旧时期的leader。

写2C的时候感觉并没有写很多东西····跑完测试会出现nextIndex过慢而fail agreement的问题。![LAB2C_1](pic\LAB2C_1.png)

需要对nextIndex增加优化，参考了[student guide](https://thesquareplanet.com/blog/students-guide-to-raft/) ，[大佬博客](https://www.jianshu.com/p/59a224fded77)

---



* If a follower does not have `prevLogIndex` in its log, it should return with `conflictIndex = len(log)` and `conflictTerm = None`.

* If a follower does have `prevLogIndex` in its log, but the term does not match, it should return `conflictTerm = log[prevLogIndex].Term`, and then search its log for the first index whose entry has term equal to `conflictTerm`.

* Upon receiving a conflict response, the leader should first search its log for `conflictTerm`. If it finds an entry in its log with that term, it should set `nextIndex` to be the one beyond the index of the *last* entry in that term in its log.

* If it does not find an entry with that term, it should set `nextIndex = conflictIndex`.
* 

### Lab3

---

因为我的服务器太渣没法跑完所有测试····虽然一开始PASS了3A但是会有warning，于是去参考了大佬的实现https://blog.csdn.net/qq_42397248/article/details/100061571，发现很多人都写了cid和seq作为请求的一部分

cid是作为client的id seq是请求的序列号，根据cid的最后请求序列号判断当前指令是否需要执行。

installRPC部分还有一些bug有几率不会通过。

###Lab4

lab4A

Join 指增加新的replica group，给GID和服务器的映射。当前的group要匀一些数据给他。

Leave 给一个list 的GID，代表这些group要离开， 新建一个configuration不包括这些group，然后把这些服务器的数据分给剩下的group。

Move 包括一些shard number 和GID代表要把这些shard转移到GID上。

Query 进行configuration的查询，也就是查询上面三个RPC的结果。

基本内容就是lab3加上shard的config。

把实验3的 分配shard应该保证每个group的shard个数尽量平均，变动的shard比较少。

维护一个每个group存储的shard。可以用堆实现，因为golang里面没有priority_queue，所以得自己实现一个堆，先用暴力的方法找最大最小，有时间我再补上。

