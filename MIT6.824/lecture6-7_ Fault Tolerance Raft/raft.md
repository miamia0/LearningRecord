##raft

###abstruct

####2.replicated state machines

为了解决什么问题:

首先他是一个consensus协议。为了解决以下的问题

1.在non-Byzantine condition下的所有情况都保证系统的安全。包括network delay,partition packet loss,  duplication ，recording.

2.只要大多数服务器都可以运行并且可以通信，它们就可以正常运行。

3.不使用timing去保证log的一致性

4.少数慢速的系统不影响全局的performance

features 

strong leader:比如log entries只会从leader到其他机器,leader election:使用随机计时的方法进行leader选举,membership changes: 。

#### 3/4.what‘s wrong with Paxos and  Why raft is better

花了一章说paxos难理解，难实现...虽然还没有看过paxos但是感受到了作者的怨念 。

设计Raft的个标：1.完整并且易于实现2.在所有条件下都是安全的，在典型的操作条件下是可用的，在通常的操作中是高效的。3.最重要的是可理解性。

后来我们意识到这种分析方法具有很强的主观性；于是我们使用了两种方法让分析变得更具通用性。第一种是关于问题分解的众所周知的方法：是否有可能，我们可以将问题分解为可以被相对独立地解释，理解并且被解决的几部分。例如，在Raft中，我们分解了leader election, log replication , safety和membership changes这几部分。

我们的第二种方法是通过减少需要考虑的状态数，尽量让系统更一致以及尽可能地减少非确定性，来简化state space。另外，log不允许存在hole，Raft限制了log之间存在不一致的可能。虽然在大多数情况下，我们都要减少不确定性，但是在某些情况下，不确定性确实提高了可理解性。特别地，随机化的方法会引入不确定性，但是通过以相同的方式处理所有可能的选择（choose any; it doesn't matter），确实减少了state space。我们就使用了随机化来减少了Raft的leader election algorithm。

#### The Raft consensus algorithm

Raft 是一个管理多台机器上的replicated log 一致性的协议。

大概流程：首先会选举一名server成为独裁的leader，其他的server都是follower，leader进行全部的管理操作包括log的更新。

优点：简单化协议， 简单化数据流。

piecewise成三个部分。

#####1.leader election:

​	在当前的leader失效后会产生新的选举。

#####2.log replication:

​	leader收到了client的log entries后，会进行复制并且更新follower的log。

#####3.safety:

​	如何保证一致性:即保证同一个index对应的log entry在每一个server上都是相同的。//todo:需要进一步解释。

##### 5.1 一个更加详细的总览

系统中的一些定义：



5.1.1server的`status`：包括leader candidate follower三种。

状态是server级别的定义。

leader：发送RPC控制整体

candidate:发送RPC竞选leader

follower:工具人，只会对candidate和leader的RPC做出回应。

![image-20191206191550370](C:\Users\12142\AppData\Roaming\Typora\typora-user-images\image-20191206191550370.png)

5.1.2 `election`: candidate通过竞争，得票超过半数的candidate成为leader，败者成为follower。

一次election的启动是system级别的定义。

5.1.3` term`: 系统运行的时间可以划分成多个term。每一个term的流程：最开始都是一次election，如果选出了leader就进入下一个term，如果没选出leader就进入下一个term并且重新进行选举。（可能选出了leader也可能没有选出leader）。

这里的term是system级别的定义。

因为term是system级别的定义所以对于server会产生以下的一些问题：1.一个server如何启动一个新的term？2.如何知道当前system处于哪一个term？或者说，server间如何进行term的同步？3.更进一步：如果发现有的server的term不对，应该怎么处理？如果发现自己的term不对，应该怎么处理？ 

1.当一个follower转成candidate状态时会发起election。

2.在每一个RPC调用过程中都会传递当前server的term信息。

![image-20191206191615000](C:\Users\12142\AppData\Roaming\Typora\typora-user-images\image-20191206191615000.png)

5.1.4server之间的交流只有RPC调用包括 RequestVoteRPC和AppendEntrieRPC。

##### 5.2 leader election

本节主要解决一下问题

1.在什么条件下会启动一个leader election？

2.如何结束一个leader election并得到哪些结果？

5.2.1 在什么条件下会启动一个leader election？

每一个server都会有一个自己的定时器，并且在初始时期都是一个follower。如果在定时器结束之前从其他server那收到了有效的RPC调用则重启定时器，如果没有收到有效的RPC的话，在定时结束了follower转变成candidate。

然后进行1.自己的term++。2.发送RequestVoteRPC。

5.2.1 如何结束一个leader election并得到哪些结果？

当candidate收到了所有的RequestVoteRPC的回信，leader election结束根据回信会有三种情况。

1)



##### 5.3 log replication

什么是commited。log entry被执行。



LOG MATCHING PROPERTY

leader改变后、

##### 5.4 safety

#####5.4.1如何保证新选出的server拥有

需要保证leader election 选出的leader已经包括之前term所有的log。即leader completeness问题。

viewstamped协议采取了在选举后把信息发送给leader的方法，这会带来额外的                                                                                                                                                                                                                                                                                                                                                开支。

raft不采取这种方法，采用在发送requestsVote的时候告知对方自己的term和index，如果follower发现他比较菜的话就拒绝投票给他。

##### 5.4.2

但是问题是，这样不是也有半数的问题吗，但是leader其实是会在过半数已经commit log后才会自己做提交，所以。

##### 5.4.3

对于完整的raft算法，我们现在可以更准确的验证[leader completeness]这个特性当前是存在的。



假设在term $T$ 时期 的leader叫做$leader_T$，他提交了log entry，但是这个log entry在未来的term上没有被store起来。接下来讨论在term$U$ ,leader为$leader_U$ 的没有store这个log entry的情况。



1.entry 在election的时候absent

2.leader_T 复制entry给了过半的server，leader_U收到了过半的server的vote

3.voter一定在给voter leader_U投票之前一定以及收到了从leader_T那commited entry 。否则它一定是拒绝了leader_T的appendentriesRPC。

4.voter在投票期间仍然保存着entry。

5.voter给leader_U投票的条件是leader_U的log是up-to-date的。

6.首先，如果voter和leader_U的last log term是相同的，leader_U的log一定至少和voter的log一样长，所以leader_U的log entry包含voter的每一个log entry。这和假设contradiction

7.另外，leader_U的last log term 必须比voter要大（>T），voter的（>=T）,由**log matching property**可得，leader_U的log一定包含commited entry。

8.在所有>T的term的leader一定包含所有的termT时期提交的entry。

9.**log matching property** 保证未来的leader也包含了不是直接提交的entry。



##### 5. 5crashes

##### 5.6


思考 1.intex和term是有关联信息的吗

2.实现上的问题，在收到rpc的时候，不同状态都会有什么反应呢？

#### 6 组内成员变化

​    在之前的小节我们都假定了组内每个成员是固定。但是实际上，会发生组员变化的情况。比如替换宕机的服务器，或者更换replication的degree。虽然更换组员可以使用离线操作的方法进行更换：先更新config文件，然后进行重启，但是这样做的话会导致重启前的这段时间内不可用。并且手动操作可能会带来一定的风险。所以打算使用自动的方式进行config的更改。并且这部分也在raft协议之内。



要保证configuration change是安全的，首先要确定在同一个term同时有两个leader。但是，立即进行原子更换每个服务器的状态是不安全的。所以cluster可以分割成两个独立的。

 

为了保证安全性，configuration的改变必须使用两相方法。比如，有些系统使用第一个phase使旧的config失效，第二个phase让新的config加入使用。在raft里cluster首先更换到过渡性的config（joint consensus）当joint consensus被提交后，系统转换到新的config。joint consensus链接了新旧的configuration。

* log entries被复制到所有服务器
* 每一个来自于configuration的server都可能成为leader。
* agreement 需要来自新旧configuration的分开的主体。

joint consensus允许独立的服务器在不同的时间更换configuration，而且，也允许在configuration change时期cluster继续进行服务。



cluster的configuration在replicated log中使用特殊的entries进行存储和交流，



#### 7 log 压缩

raft的log在一般的操作将更多的客户端请求包括在内，但是在实际的系统中，log不能无限制的增长，需要进行压缩。
snapshot是进行压缩的最简单的方式。在snapshotting中，当前系统的状态被整个写到一个stable storage的snapshot中。







