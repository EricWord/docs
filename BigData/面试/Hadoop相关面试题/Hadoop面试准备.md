[TOC]

# 1. MapReduce的工作流程(掌握程度3分)

**回答1：**

1. 在客户端执行submit()方法之前，会先去获取一下待读取文件的信息
2. 将Job提交给yarn，这时候会带着3个信息过去(job.split(文件的切片信息),jar.job.xml)
3. Yarn会根据文件的切片信息去计算将要启动的MapTask的数量，然后去启动MapTask
4. MapTask会调用InputFormat()方法去HDFS上面读取文件,InputFormat()方法会再去调用RecordRead()方法，将数据以行首字母的偏移量为key,一行数据为value传给mapper()方法
5. Mapper()方法做一些逻辑处理后，将数据传到分区方法中，对数据进行一个分区标注后，发送到环形缓冲区中
6. 环形缓冲区默认的大小是100M，达到80%的阈值将会溢写
7. 在溢写之前会做一个排序的动作，排序的规则是按照key进行字典序排序，排序的手段是快排
8. 溢写会产生出大量的溢写文件，会再次调用merge()方法，使用归并排序，默认10个溢写文件合并成一个大文件
9. 也可以对溢写文件做一次localReduce也就是combiner的操作，但前提是combiner的结果不能对最终的结果有影响
10. 等待所有的MapTask结束之后，会启动一定数量的ReduceTask
11. ReduceTask会启动拉取线程到map端拉取数据，拉取到的数据会先加载到内存中，内存不够会写到磁盘里，等待所有的数据数据拉取完毕会将这些输出再进行一次归并排序
12. 归并后的文件会再次进行一次分组的操作，然后将数据以组为单位发送到reduce()方法
13. reduce()方法做一些逻辑判断后，最终调用OutputFormat()方法，OutputFormat()会再去调用RecordWrite()方法将数据以KV的形式写出到HDFS上

**回答2：**

​        1.客户端将每个block块切片（逻辑切分），每个切片都对应一个map任务，默认一个block块对应一个切片和一个map任务，split包含的信息：分片的元数据信息，包含起始位置，长度，和所在节点列表等

　　2.map按行读取切片数据，组成键值对，key为当前行在源文件中的字节偏移量，value为读到的字符串

　　3.map函数对键值对进行计算，输出<key,value,partition（分区号）>格式数据，partition指定该键值对由哪个reducer进行处理。通过分区器，key的hashcode对reducer个数取模。
　　4.map将kvp写入环形缓冲区内，环形缓冲区默认为100MB，阈值为80%，当环形缓冲区达到80%时，就向磁盘溢写小文件，该小文件先按照分区号排序，区号相同的再按照key进行排序，归并排序。溢写的小文件如果达到三个，则进行归并，归并为大文件，大文件也按照分区和key进行排序，目的是降低中间结果数据量（网络传输），提升运行效率
　　5.如果map任务处理完毕，则reducer发送http get请求到map主机上下载数据，该过程被称为洗牌shuffle
　　6.可以设置combinclass（需要算法满足结合律），先在map端对数据进行一个压缩，再进行传输，map任务结束，reduce任务开始
　　7.reduce会对洗牌获取的数据进行归并，如果有时间，会将归并好的数据落入磁盘（其他数据还在洗牌状态）
　　8.每个分区对应一个reduce，每个reduce按照key进行分组，每个分组调用一次reduce方法，该方法迭代计算，将结果写到hdfs输出

 **洗牌阶段**

　　1.copy:一个reduce任务需要多个map任务的输出，每个map任务完成时间很可能不同，当只要有一个map任务完成，reduce任务立即开始复制，复制线程数配置mapred-site.xml参数“mapreduce.reduce.shuffle.parallelcopies"，默认为5.

　　2.copy缓冲区：如果map输出相当小，则数据先被复制到reduce所在节点的内存缓冲区大小配置mapred-site.xml参数“mapreduce.reduce.shuffle.input.buffer.percent”，默认0.70），当内存缓冲区大小达到阀值（mapred-site.xml参数“mapreduce.reduce.shuffle.merge.percent”，默认0.66）或内存缓冲区文件数达到阀值（mapred-site.xml参数“mapreduce.reduce.merge.inmem.threshold”，默认1000）时，则合并后溢写磁盘。
　　3.sort：复制完成所有map输出后，合并map输出文件并归并排序
　　4.sort的合并：将map输出文件合并，直至≤合并因子（mapred-site.xml参数“mapreduce.task.io.sort.factor”，默认10）。例如，有50个map输出文件，进行5次合并，每次将10各文件合并成一个文件，最后5个文件。



# 2. HDFS分布式存储工作机制

## 2.1 HDFS的重要特性

**First.** HDFS是一个文件系统，用于存储和管理文件，通过统一的命名空间（类似于本地文件系统的目录树）。是分布式的，服务器集群中各个节点都有自己的角色和职责。

　　**Then.**

　　1.HDFS中的文件在物理上是**分块存储（block）**，块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，之前的版本中是64M。

　　2.HDFS文件系统会给客户端提供一个**统一的抽象目录树**，客户端通过路径来访问文件，形如：hdfs://namenode:port/dir-a/dir-b/dir-c/file.data

　　3.**目录结构及文件分块位置信息(元数据)**的管理由namenode节点承担，namenode是HDFS集群主节点，负责维护整个hdfs文件系统的目录树，以及每一个路径（文件）所对应的数据块信息（blockid及所在的datanode服务器）

　　4.文件的各个block的存储管理由datanode节点承担，datanode是HDFS集群从节点，每一个block都可以在多个datanode上存储多个副本（副本数量也可以通过参数设置dfs.replication，默认是3）

　　5.Datanode会定期向Namenode汇报自身所保存的文件block信息，而namenode则会负责保持文件的副本数量，HDFS的内部工作机制对客户端保持透明，客户端请求访问HDFS都是通过向namenode申请来进行。

　　6.HDFS是设计成适应一次写入，多次读出的场景，且不支持文件的修改。需要频繁的RPC交互，写入性能不好。



## 2.2 **HDFS写数据分析**

**1.概述**

 　客户端要向HDFS写数据，首先要跟namenode通信以确认可以写文件并获得接收文件block的datanode，然后客户端按顺序将文件逐个block传递给相应datanode，并由接收到block的datanode负责向其他datanode复制block的副本。

**2.写数据步骤详解**

![image-20210219112421088](./images/1.png)

　　1）客户端向namenode发送上传文件请求，namenode对要上传目录和文件进行检查，判断是否可以上传，并向客户端返回检查结果。

　　2）客户端得到上传文件的允许后读取客户端配置，如果没有指定配置则会读取默认配置（例如副本数和块大小默认为3和128M，副本是由客户端决定的）。向namenode请求上传一个数据块。

　　3）namenode会根据客户端的配置来查询datanode信息，如果使用默认配置，那么最终结果会返回同一个机架的两个datanode和另一个机架的datanode。这称为“机架感知”策略。

> 机架感知：HDFS采用一种称为机架感知(rack-aware)的策略来改进数据的可靠性、可用性和网络带宽的利用率。大型HDFS实例一般运行在跨越多个机架的计算机组成的集群上，不同机架上的两台机器之间的通讯需要经过交换机。在大多数情况下，同一个机架内的两台机器间的带宽会比不同机架的两台机器间的带宽大。通过一个机架感知的过程，Namenode可以确定每个Datanode所属的机架id。一个简单但没有优化的策略就是将副本存放在不同的机架上。这样可以有效防止当整个机架失效时数据的丢失，并且允许读数据的时候充分利用多个机架的带宽。这种策略设置可以将副本均匀分布在集群中，有利于当组件失效情况下的负载均衡。但是，因为这种策略的一个写操作需要传输数据块到多个机架，这增加了写的代价。在大多数情况下，副本系数是3，HDFS的存放策略是将一个副本存放在本地机架的节点上，一个副本放在同一机架的另一个节点上，最后一个副本放在不同机架的节点上。这种策略减少了机架间的数据传输，这就提高了写操作的效率。机架的错误远远比节点的错误少，所以这个策略不会影响到数据的可靠性和可用性。于此同时，因为数据块只放在两个（不是三个）不同的机架上，所以此策略减少了读取数据时需要的网络传输总带宽。在这种策略下，副本并不是均匀分布在不同的机架上。三分之一的副本在一个节点上，三分之二的副本在一个机架上，其他副本均匀分布在剩下的机架中，这一策略在不损害数据可靠性和读取性能的情况下改进了写的性能

4）客户端在开始传输数据块之前会把数据缓存在本地，当缓存大小超过了一个数据块的大小，客户端就会从namenode获取要上传的datanode列表。之后会在客户端和第一个datanode建立连接开始流式的传输数据，这个datanode会一小部分一小部分（4K）的接收数据然后写入本地仓库，同时会把这些数据传输到第二个datanode，第二个datanode也同样一小部分一小部分的接收数据并写入本地仓库，同时传输给第三个datanode，依次类推。这样逐级调用和返回之后，待这个数据块传输完成客户端后告诉namenode数据块传输完成，这时候namenode才会更新元数据信息记录操作日志。

5）第一个数据块传输完成后会使用同样的方式传输下面的数据块直到整个文件上传完成。

**细节：**

　　a.请求和应答是使用RPC的方式，客户端通过ClientProtocol与namenode通信，namenode和datanode之间使用DatanodeProtocol交互。在设计上，namenode不会主动发起RPC，而是响应来自客户端或 datanode  的RPC请求。客户端和datanode之间是使用socket进行数据传输，和namenode之间的交互采用nio封装的RPC。

　　b.HDFS有自己的序列化协议。

　　c.在数据块传输成功后但客户端没有告诉namenode之前如果namenode宕机那么这个数据块就会丢失。

　　d.在流式复制时，逐级传输和响应采用响应队列来等待传输结果。队列响应完成后返回给客户端。

　　c.在流式复制时如果有一台或两台（不是全部）没有复制成功，不影响最后结果，只不过datanode会定期向namenode汇报自身信息。如果发现异常namenode会指挥datanode删除残余数据和完善副本。如果副本数量少于某个最小值就会进入安全模式。

> 安全模式：Namenode启动后会进入一个称为安全模式的特殊状态。处于安全模式的Namenode是不会进行数据块的复制的。Namenode从所有的  Datanode接收心跳信号和块状态报告。块状态报告包括了某个Datanode所有的数据块列表。每个数据块都有一个指定的最小副本数。当Namenode检测确认某个数据块的副本数目达到这个最小值，那么该数据块就会被认为是副本安全(safely  replicated)的；在一定百分比（这个参数可配置）的数据块被Namenode检测确认是安全之后（加上一个额外的30秒等待时间），Namenode将退出安全模式状态。接下来它会确定还有哪些数据块的副本没有达到指定数目，并将这些数据块复制到其他Datanode



## 2.3 HDFS读数据分析

**1.概述**

 　客户端将要读取的文件路径发送给namenode，namenode获取文件的元信息（主要是block的存放位置信息）返回给客户端，客户端根据返回的信息找到相应datanode逐个获取文件的block并在客户端本地进行数据追加合并从而获得整个文件。

**2.读数据步骤详解**

![image-20210219112639188](./images/2.png)

​        1）客户端向namenode发起RPC调用，请求读取文件数据。

　　2）namenode检查文件是否存在，如果存在则获取文件的元信息（blockid以及对应的datanode列表）。

　　3）客户端收到元信息后选取一个网络距离最近的datanode，依次请求读取每个数据块。客户端首先要校检文件是否损坏，如果损坏，客户端会选取另外的datanode请求。

　　4）datanode与客户端简历socket连接，传输对应的数据块，客户端收到数据缓存到本地，之后写入文件。

　　5）依次传输剩下的数据块，直到整个文件合并完成。

> 它会检验从Datanode获取的数据跟相应的校验和文件中的校验和是否匹配，如果不匹配，客户端可以选择从其他Datanode获取该数据块的副本



## 2.4 HDFS删除数据分析

HDFS删除数据比较流程相对简单，只列出详细步骤:

　　1）客户端向namenode发起RPC调用，请求删除文件。namenode检查合法性。

　　2）namenode查询文件相关元信息，向存储文件数据块的datanode发出删除请求。

　　3）datanode删除相关数据块。返回结果。

　　4）namenode返回结果给客户端。

> 　当用户或应用程序删除某个文件时，这个文件并没有立刻从HDFS中删除。实际上，HDFS会将这个文件重命名转移到/trash目录。只要文件还在/trash目录中，该文件就可以被迅速地恢复。文件在/trash中保存的时间是可配置的，当超过这个时间时，Namenode就会将该文件从名字空间中删除。删除文件会使得该文件相关的数据块被释放。注意，从用户删除文件到HDFS空闲空间的增加之间会有一定时间的延迟。只要被删除的文件还在/trash目录中，用户就可以恢复这个文件。如果用户想恢复被删除的文件，他/她可以浏览/trash目录找回该文件。/trash目录仅仅保存被删除文件的最后副本。/trash目录与其他的目录没有什么区别，除了一点：在该目录上HDFS会应用一个特殊策略来自动删除文件。目前的默认策略是删除/trash中保留时间超过6小时的文件。将来，这个策略可以通过一个被良好定义的接口配置。
>
> 　　当一个文件的副本系数被减小后，Namenode会选择过剩的副本删除。下次心跳检测时会将该信息传递给Datanode。Datanode遂即移除相应的数据块，集群中的空闲空间加大。同样，在调用setReplication API结束和集群中空闲空间增加间会有一定的延迟。

## 2.5 NameNode元数据管理原理分析

**1.概述**

　　首先明确namenode的职责：响应客户端请求、管理元数据。

　　namenode对元数据有三种存储方式：

　　内存元数据(NameSystem)

　　磁盘元数据镜像文件

　　数据操作日志文件（可通过日志运算出元数据）

　　细节：HDFS不适合存储小文件的原因，每个文件都会产生元信息，当小文件多了之后元信息也就多了，对namenode会造成压力。

　　**2.对三种存储机制的进一步解释**

　　内存元数据就是当前namenode正在使用的元数据，是存储在内存中的。

　　磁盘元数据镜像文件是内存元数据的镜像，保存在namenode工作目录中，它是一个准元数据，作用是在namenode宕机时能够快速较准确的恢复元数据。称为fsimage。

　　数据操作日志文件是用来记录元数据操作的，在每次改动元数据时都会追加日志记录，如果有完整的日志就可以还原完整的元数据。主要作用是用来完善fsimage，减少fsimage和内存元数据的差距。称为editslog。

　　**3.checkpoint机制分析**

　　因为namenode本身的任务就非常重要，为了不再给namenode压力，日志合并到fsimage就引入了另一个角色secondarynamenode。secondarynamenode负责定期把editslog合并到fsimage，“定期”是namenode向secondarynamenode发送RPC请求的，是按时间或者日志记录条数为“间隔”的，这样即不会浪费合并操作又不会造成fsimage和内存元数据有很大的差距。因为元数据的改变频率是不固定的。

　　每隔一段时间，会由secondary namenode将namenode上积累的所有edits和一个最新的fsimage下载到本地，并加载到内存进行merge（这个过程称为checkpoint）。

![image-20210219112821929](./images/3.png)

​        1）namenode向secondarynamenode发送RPC请求，请求合并editslog到fsimage。

　　2）secondarynamenode收到请求后从namenode上读取（通过http服务）editslog（多个，滚动日志文件）和fsimage文件。

　　3）secondarynamenode会根据拿到的editslog合并到fsimage。形成最新的fsimage文件。（中间有很多步骤，把文件加载到内存，还原成元数据结构，合并，再生成文件，新生成的文件名为fsimage.checkpoint）。

　　4）secondarynamenode通过http服务把fsimage.checkpoint文件上传到namenode，并且通过RPC调用把文件改名为fsimage。

　　namenode和secondary namenode的工作目录存储结构完全相同，所以，当namenode故障退出需要重新恢复时，可以从secondary  namenode的工作目录中将fsimage拷贝到namenode的工作目录，以恢复namenode的元数据。

　　关于checkpoint操作的配置：

> dfs.namenode.checkpoint.check.period=60  #检查触发条件是否满足的频率，60秒
>
> dfs.namenode.checkpoint.dir=file://${hadoop.tmp.dir}/dfs/namesecondary
>
> \#以上两个参数做checkpoint操作时，secondary namenode的本地工作目录
>
> dfs.namenode.checkpoint.edits.dir=${dfs.namenode.checkpoint.dir}
>
> dfs.namenode.checkpoint.max-retries=3 #最大重试次数
>
> dfs.namenode.checkpoint.period=3600 #两次checkpoint之间的时间间隔3600秒
>
> dfs.namenode.checkpoint.txns=1000000 #两次checkpoint之间最大的操作记录

editslog和fsimage文件存储在$dfs.namenode.name.dir/current目录下，这个目录可以在hdfs-site.xml中配置的。这个目录下的文件结构如下：

![image-20210219112929824](./images/4.png)

包括edits日志文件（滚动的多个文件），有一个是edits_inprogress_*是当前正在写的日志。fsimage文件以及md5校检文件。seen_txid是记录当前滚动序号，代表seen_txid之前的日志都已经合并完成

> 　　$dfs.namenode.name.dir/current/seen_txid非常重要，是存放transactionId的文件，format之后是0，它代表的是namenode里面的edits_*文件的尾数，namenode重启的时候，会按照seen_txid的数字恢复。所以当你的hdfs发生异常重启的时候，一定要比对seen_txid内的数字是不是你edits最后的尾数，不然会发生重启namenode时metaData的资料有缺少，导致误删Datanode上多余Block的信息。

# 3. HadoopHA架构原理











# 4. Yarn的资源调度工作机制









# 5. Hadoop的组成结构







# 6. MapReduce的shuffle过程





# 7. Hadoop的垃圾回收机制







# 8. Hadoop离线查看小文件





# 9. MapReduce为什么不直接Map或者Reduce







# 10. HDFS小文件怎么处理的？







# 11. Map任务和Reduce在哪运行的







# 12. Hadoop文件大小怎么切分





