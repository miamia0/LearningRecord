#### 二、overview

对于一个系统的使用情况的假设

1.系统需要做什么：系统应该能够检测到错误，有高容错性，并及时修正错误。

2.系统存储着什么：系统存储适量的大文件 ，我们假设会有几百万的大于100MB的文件。当然GB级别的文件也很common。对于小文件的存储也是需要支持的，但是没有必要对其 的存储查询进行优化。

3.系统的读操作功能要求:包括两种读取操作1）large straming reads 2) small random reads 。

4.系统的写操作功能要求： 通常操作的大小与读取相似。一旦完成写入，文件几乎不会再被修改。系统也会支持小型随机写入操作，但是效率不会很高。（比如说百度网盘每次修改感觉也都是先把文件放到本地然后修改后再重新上传） 

5.多client同时写同一个文件的问题:producer-consumer队列

6.读写操作的响应时间和数据处理速度的trade-off：GFS在响应时间和数据处理中更看重批量处理数据的速度， 持续的bandwidth比low lantency更为重要。

(//todo这里的批量数据处理应该代表什么样子的数据处理)

#####2.2提供的interface

包括 create delete open close read write操作

其他功能: 

* snapshot：

* record  append。多个客户端可以同时给同一个文件后append数据。这种设计主要是为了在大规模分布式计算中的运用。

##### 2.3架构

 整个架构包括一个*master*和多个*chunkserver*构成，他们都可以被多个客户端访问。 

文件被分为大小相同的chunks，每一个chunk都有一个唯一且不变的id(64bit大小)在chunk创建的时候被master机器分配的。为了实现reliability，默认每一个chunk都会创建三个replicas， 当然用户可以指定不同的file namespace中的region具有不同的replication的级别。 （这里大概是指有些部分的文件可以有更高的replication的级别从而replica能够有更多份）。

master 存储着文件系统的metadata，包括namespace，access control information，mapping from files to chunks，管理 chunks所在的位置(在哪个chunk server上) 。 也包括chunk lease 的管理和孤立chunk的辣鸡回收。对于chunkserver会使用heartbeat message 来进行管理和了解server情况。

​	client访问master获取metadata，但是获取文件数据通信都将直接访问chunkserver 。因为对文件访问并不提供POSIX的API。( Portable Operating System Interface of UNIX )，                  所以不需要hook into linxu的vnode layer(这里的vnode layer指的是啥)。

​	文件cache问题,client和chunkserver都不做file data的cache。因为client多数应用使用的文件都很巨大，输出大小超过cache的范围。没有缓存让客户端和系统都变得比较简单，因为不需要考虑cache的同步需求。chunkserver直接使用linux buffer cache。

#####2.4 只有一个人的孤单master

 ![img](http://dl.iteye.com/upload/attachment/0082/6881/e7cb7cc8-c001-3268-8521-18e0bc4cf626.jpg) 

   让我们解释一下一个简单的read操作的交互过程，参见图1.首先，使用固定chunk size，client将应用指定的文件名称和byte offset转换成文件系统的一个chunk索引（filename 和byte offset可以最终确定为某个file中的特定chunk信息）。然后，它（client）向master发送一个包含文件名和chunk索引的请求，master响应给client相应的chunk handle和replicas的位置。Client使用fileName和chunk index作为key来缓存这些信息。(问题:对于文件中超过chunksize溢出的部分应该怎么存储呢？)client然后就会其中一个replica发送request。request包括chunk handle和byte range，如果还需要读取相同的chunk的话在缓存信息过期前或者读取的文件被重新打开前就不需要和master进行新的通信了。为了减少与master的通信次数，client每次会在和master的通讯中发送的请求中请求多个chunk的位置。

#####2.5 chunksize

这里选择64MB作为chunksize

为什么感觉这一节就是在狡辩==大的chunkize好处是 显而易见的，对于大的文件可以加快传输速度。但是对于小文件可能会有hotspot问题。

在hotspots尚未被care的时候，GFS开始使用的是批量队列系统：一个可执行文件作为一个single-chunk的文件被写入到GFS，然后同时在多个机器上启动。存储这个可执行文件（executeable）的几个chunkservers在数个请求同时访问时可能会过载。通过使用更高的复制因子（higher replication factor）存储这个执行文件，使被处理队列系统错开应用启动的时间。在这种情况下，一个潜在的解决办法就是允许客户端可以从其他客户端读取数据。
???**完全不明白这里的更高的复制因子是在干啥?**



##### 2.6metadata

存储了三种类型的metadata: file 和chunk的namespace ，和他们之间的mapping，和他们的replica，

存储chunk的时机 master启动时加入所有的或者一个chunkserver加入时进行collect，(断开时是不是要清空？)。

可以从另一个方向来认识这种设计，chunkserver上是否持有chunk，chunkserver具有最终决定权。那就没有必要在master中再持久存储一份视图数据，而且chunkserver的一些error会导致chunk信息的丢失（硬盘损坏或者不可用），甚至有的操作会重命名chunkserver。所以采用  master不会持久存储chunkserver所持有的replicas信息，它（master）仅仅是在启动后去各个chuankserver上poll这些数据，然后会保持自己所持有chunk信息的更新 



##### 2.7 consistency model




####三、系统中的交互设计

因为GPS的设计理念是减少master对于所有操作的参与。

这里的交互包括master，client，chunkserver的交互。操作包括data mutation，atomic record append，snapshot。

//todo atomic record append 和snapshot的名词解释

#####3.1 leases顺序与mutation顺序

mutation操作在这里指的是







客户端询问master哪一个chunkserver当前是lease server





### 四 master节点的有关操作



master节点1.管理所有数据(chunk replicas)2.判定文件chunk存放的位置3.创建新的chunk 4.协调各种系统中的行为从而使chunks5.回收不再使用的空间

