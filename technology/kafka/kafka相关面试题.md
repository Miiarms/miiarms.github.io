##### （1）kafka实现高吞吐量的核心机制

- os cache写：先将数据写入到os cache中，然后通过同步或者异步刷盘
- 磁盘顺序写：写文件仅仅是追加数据到文件末尾，几乎不会进行随机写，即便是机械硬盘，性能也不会比固态硬盘低太多
- 零拷贝：非零拷贝下，数据会先读取到os cache中再复制到用户进程，再写入到socket cache中进行发送；零拷贝的话，直接读取到os cache，通过linux的sendFile()输出到网卡进行发送
- 页缓存技术：前提是kafka的消息被读很实时的话，就会直接从os cache中读取需要的数据，和es很像，如果绝大多数的数据能被放入到os cache中，那性能是非常高的



##### （2）Broker是无状态的

- 类似于topic、partition这些元数据信息是不会记录在broker上的，而是会记录在zk上
- 所有对于broker上线、下线、controller选举，都是通过zk来实现



##### （3）Kafka Broker集群进行Controller选举以及partition分配

- Broker启动与Controller选举：第一个将自身信息写入到Controller节点的机器被选为Controller，其他所有Broker会缓存Controller的ip、port等信息
- 创建topic以及partition的分配方案：客户端访问任何一台Broker就知道哪台是Controller，向Controller发送创建topic请求，例如partition=10，replica=3，Controller指定好partition分配方案之后，就将方案信息写入到zk上
- Broker获取到partition分配方案：broker监听到自己的分配方案，创建对应的内存、物理结构；数据文件只有写入第一条数据的时候才会被创建



##### （4）Producer往kafka写入消息

- Producer访问任意Broker获取partition分配信息（包含了leader partition在哪几台机器上，是有多个的）
- 如果消息设置了key，就取取key.hashCode%可用broker数量；没有设置key的话，就用一个AtomicInteger，进行RoundRobin，选出要写入的leader partition
- 将消息按照kafka的二进制协议格式转化成ByteBuffer，并放入到缓存中当前可用的一个Batch里（并不是写一条发送一条，所以这里可能还是会丢数据）
- 当某一个Batch达到一定长度或者存活时间大于了阈值，最后由Sender线程，将整个Batch发送到对应的Broker上（默认batch为16k，阈值100ms）



##### （5）leader partition所在的Broker接收到消息之后进行处理

- 假设发送的消息offset=362，当前partition的leo=360，hw=350
- 首先判断当前的log文件的大小是否已经大于1G，或者创建时间已经过去了7天；是则创建新的log和index文件，一个partition就对应一个文件夹，否则返回当前segment提供写入
- 将二进制数据连同一些前置字段写入到当前segment对应的log文件中，判断当前写入数据距离上一个建立index稀疏索引的数据相加是否大于了4KB，是的话，则需要再次建立一条心的稀疏索引记录
- 将这条消息放入待同步队列中，等待Follower所在的Broker来拉取，并更新本地leo=362



##### （6）follower所在的broker发送fetch请求

- 假设当前follower的leo=350，hw=340，那么发送的请求会将leo=351发送给leader broker
- Leader Broker收fetch请求之后，找到broker对应的队列，此时应该有12条消息，最大的offset=362，将这12条消息连同HW=350一起返回给follower broker
- follower收到返回的信息，将消息按照二进制协议写入到本地磁盘中，其过程跟5中一样，然后更新本地leo=363
- 对比本地leo=363和leader返回的hw=350，仍然取最小的值350，作为本地hw
- 将响应返回给leader broker，leader接收到满足最少ack数量的follower的返回之后，才允许返回发送消息成功的响应给生产者，否则发送失败



##### （7）Consumer如何加入到group上

- Consumer会先选出Coordinator节点：Consumer发送GroupCoordinatorRequest到任意Broker上，Broker节点根据groupId的hash值，来与consumer_offsets这个topic的partition数量取模，得出某个partition具体编号，假设groupId.hashCode=5024，consumer_offsets这个topic的partition数量默认为50，那么被选中的partition编号就为24，遍历__consumer_offsets所有的partition，找出那个编号为24的partition对应的Broker信息，这个Broker就是Coordinator
- Consumer进行join_group操作：Consumer发送join_group指令到Coordinator节点，Coordinator节点收到请求，如果group尚未创建，那么就会通过GroupManager创建对应的group，选择consumer leader的算法也十分地粗暴，就是第一个加入到group的节点；Coordinator节点返回join_group响应，其中最重要的就是包含发送请求的Consumer是否是leader、以及group中的member列表
- Consumer Leader指定partition assignment：Consumer Leader收到member列表，通过内置的分配器，例如RangeAssignor类进行分配，RangeAssignor的分配策略也十分粗暴，将所有的leader partition，按照范围进行划分，如果除不尽的话，全部由第一个consumer来消化
- Consumer Leader发送sync_group指令：将步骤3中指定的partition assignment方案作为sync_group请求发送给Coordinator节点，Coordinator节点接收到partition_assignment，再将其下发给group里的所有member，每一个member收到响应后，将partition_assignment里指定的方案缓存到本地



##### （8）Consumer如何开始消费

- Consumer根据partition_assignment，封装一组fetch request，每一个fetch request对应一个Broker
- Consumer将一个Broker对应的fetch request发送到对应的Broker上，Broker收到请求之后，开始从磁盘上读取数据
- Consumer根据请求里的partition和offset，开始从磁盘上加载数据，先从index文件里找出最接近offset的那条稀疏索引，根据稀疏索引里的文件位置，开始从log文件中读取，直至找到offset的那条记录，将offset对应的记录以及后续写入的消息读取，返回给消费者



##### （9）Consumer进行自动或者手动commit

- 这个过程很简单，就是发送commit offset请求到对应的Broker上
- Broker将committed offset消息写入到__consumer_offset这个topic上
- 后续consumer继续发送fetch request，都只能拉取比__consumer_offset上的offset大的消息



##### （10）采用分段机制保存日志

- 每一个partition就对应一个目录，名称类似于：order-topic-0、order-topic-1
- 每个分区里面就是很多的log segment file，一个segment代表一个log文件和一个index文件，前者是日志文件、后者是索引文件



##### （11）Kafka索引文件与log文件

- 采用二分法从索引文件中进行搜索：对默认4KB长度的数据建立一条索引，就是所谓的“稀疏索引”，将索引记录读进内存，就可以使用二分法去查找，每条索引里边存有offset和对应的消息的起始文件位置，另外如果需要通过时间去查询的话，那么就可以通过.timeindex时间索引文件进行检索
- log文件对于offset的检索：通过索引文件拿到最接近的offset对应的消息的log文件日志位置，从该位置开始读取，按照二进制协议格式，进行读取，直到读到目标offset对应的记录为止
- 日志的定期清理与回放：默认保存7天的数据，通过log.retention.hours、log.retention.days来指定，典型的场景，可以将例如3天的数据，重新定位消费offset之后，将其全部重放一遍



##### （12）时间轮算法推演

- 例如一个延迟352ms的延迟任务被放入到时间轮中，每一级时间轮跨度*10
- 例如是1ms、10ms、100ms，那么如何去推演这个352ms的任务最终如何落入到第一级，2ms这个位置去被执行



##### （13）kafka全链路数据丢失风险分析

- 消息还在producer缓冲区：直接就生产服务就挂了，此时数据就丢失了
- acks = 1：leader写入成功了，就可以认为是成功，如果数据刚刚写入leader成功，follower还没同步数据，此时leader宕机必然会导致数据的丢失
- acks = -1：leader和follower都写成功了，此时任何一个副本宕机，都不会导致数据的丢失，如果min.insync.replicas = 1，此时如果follower先宕机了，导致ISR里就一个leader了，leader写入成功，就算成功了，结果leader也挂了，此时数据还是会丢失的
- auto ack：poll到了数据，还没来得及处理，自动提交offset，此时kafka就认为已经成功处理了这批数据，客户端宕机的话，数据就丢失了



##### （14）线上数据总是莫名其妙出现重复

- 生产端是有重试机制：有一些网络抖动，在底层的网络环节，其实消息发送出去了，结果收到了一个NetworkException，此时会重发消息
- 消费端重复的问题：很频繁，如果你是用自动offset提交，每次重启consumer服务的时候，一定会重复消费消息



##### （15）MySQL binlog数据同步顺序为何不能出错

- 生产者发送消息的重试造成：导致乱序的原因还是重试，<NodeId, Deque\<Request>>，最多允许同时发送出去5个请求，忍受5个请求都发出去但是没有响应，此时处于重试机制，请求3最先重试，那么就会造成乱序
- 消费者消费多partition的原因：同一个顺序的DDL操作对应的binlog日志，会发送给不通的partition，如果这些partition都被不通的消费者给消费了，那么就会造成乱序，毕竟每一个消费者消费的时间不确定



##### （16）线上消息队列百万消息积压怎么处理

- 比如说他是依赖于MySQL、NoSQL之类的系统，但是现在就是说MySQL他的性能突然出现了很大的问题
- 比如一个表有几千万条数据，有个同学在高峰期做了一个DDL，导致对数据库的增删改查非常的慢，性能急剧下降几十倍，瞬间导致kafka里大量的积压了很多很多的消息，几百万条消息积压在里面



##### （17）如何配合分布式事务方式实现消息事务支持

- 可靠消息最终一致性的事务方案
- 这套方案是基于MQ来实现的，可靠消息服务，上游服务的本地事务成功了，必须保证消息投递出去到MQ上去，可以由MQ自己来支持和实现，RocketMQ就实现了一套机制
- 提供了事务相关的支持，数据库事务 + 消息，封装在一个事务里，数据库事务成功了，消息也必须投递成功，如果要跟分布式事务来整合的话，一般用的不是Kafka，RocketMQ来支持就最好



##### （18）消息的过期时间（TTL）如何实现

发现消息达到ttl时间，直接commit即可；或者消费下来，写入一个专门的topic来特殊处理即可



##### （19）如何实现延迟队列的效果

如果发现下游消费者压力太大，可以选择发送延迟消息



##### （20）不同的消息如何实现多优先级队列

发送消息可以指定优先级，例如一些vip客户的数据，被优先消费



##### （21）针对链路故障的死信队列应该如何实现

进入某个topic的一条消息，无法被消费者正确地处理，那么就进入一个特殊的队列



##### （22）消费服务故障场景下的重试队列

结合TTL，死信队列，针对一些异常故障的场景来进行处理



##### （23）下游数据计算错误时如何进行消息回溯

使用kafka的重定向offset的机制，将处理后的结果删除或者回退，寻找一个时间点，将数据全部重回一遍即可



##### （24）不同的业务数据如何实现消息路由

对同一个topic里的数据实现消息路由，同一个topic里可能也细分为不同的数据，对消息打上不同的标签，消息路由



##### （25）对消息质量是否正常进行监控

需要对所有的消息去进行一个监控汇总，每天一共流转过去多少条消息，10亿条，其中链路完整流转完整的消息占比多少，链路不完整处理错误的占比多少



##### （26）offset、leo、hw各代表什么意思？

- offset指消息的唯一编号
- leo指当leader收到一条消息之后，会将当前leader的leo设置为该消息的offset + 1
- 当一条消息被绝大多数follower同步之后，会将hw + 1
- Consumer是只能够消费到HW之前的数据，offset > HW的消息是不会被消费到的



##### （27）leo和hw是如何更新的

- 假设当前leader partition的leo=362，hw=350，follower partition的leo=351，hw=340，就说明leader有12条数据需要同步
- 第一次fetch，follower partition发送fetch request，带上leo=351；leader partition将12条消息返回给follower，并带上hw=350；follower收到响应，取leader hw和自己的leo最小值为当前hw，所以设置hw=350，同时更新自己的leo=362
- 第二次fetch，follower partition发送fetch request，带上leo=362；leader此时会把自身的hw更新为361，并返回给follower；follower收到响应之后，取响应里的hw=361和自己的leo=362中的最小值，更新为本地的hw值，也就是351



##### （28）高水位机制可能导致leader切换时发生数据丢失问题

- 这种情况是当min.insync.replicas为1的时候
- 那么当一条消息写入到leader之后就立即返回，那么当leader宕机，而follower成为leader之后，原leader加入到新leader上进行数据同步，会将这条数据删除



##### （29）高水位机制可能导致leader切换时发生数据不一致问题

- 和数据丢失的情况一样，发生在min.insync.replicas为1的时候
- 将数据写入leader之后，生产者即返回成功；follower进行fetch，将这条数据写入到本地，并更新本地leo，这时候发生leader宕机，而follower成为新leader
- 新leader不会截断这条数据，而当原leader成为follower之后，与新leader进行数据同步，发现自己的leo大于leader的leo，会将这条消息截断掉



##### （30）epoch机制解决高水位机制弊端（对每一次leader选举新增了一个版本号的概念）

- Leader Epoch解决数据丢失问题，假设发生了min.insync.replicas=1情况下数据丢失的问题，那么原leader重启为follower之后，还是用原来的epoch号去同步消息，发现epoch比新leader要小，所以不截断数据
- Leader Epoch解决数据不一致的问题，这个很容易理解，新leader出现epoch<1, 2><2， 4>，而原来的leader变成follower之后的epoch<1, 3>，那么follower就知道offset=3的数据不应该存在，进行截取



##### （31）ISR列表机制

- 每一个leader partition都维护了一个ISR列表，保存它的follower partition列表和对应的leo
- 新版本中，follower落后于leader最长达到replica.lag.time.max.ms指定的时间阈值，就将这个follower剔除掉



##### （32）老版本中依靠follower落后于leader的消息条数来剔除follower的弊端

- 这个很简单，如果瞬时并发量太大，就会造成误剔除



##### （33）一般导致follower跟不上的主要情况

- follower所在机器的性能变差，比如说网络负载过高，IO负载过高，CPU负载过高，机器负载过高
- follower所在的broker进程卡顿，常见的就是fullgc问题，kafka自己本身对jvm的使用是很有限的，一般不怎么在自己的内存里维护过多的数据，主要是依托os cache（缓存）来提高读和写的性能的
- kafka是支持动态调节副本数量的，如果动态增加了partition的副本，就会增加新的follower，此时新的follower会拼命从leader上同步数据，但是这个是需要过程的



##### （34）Reactor线程模型

- acceptor线程：每个broker上都有一个acceptor线程，来监听每个socket连接的接入，acceptor线程会采用轮询的机制，给新接入的socket分配Processor线程
- Processor线程：每个broker上有默认3个processor线程，通过num.network.threads指定，该线程通过nio读取各种网络事件，并交给工作线程去处理
- 请求队列：processor线程会负责把请求放入一个broker全局唯一的请求队列中，工作线程负责去处理
- KafkaRequestHandler线程池：这个就是工作线程池，用来处理各种复杂的业务逻辑，默认数量为8个
- 响应队列：每一个Processor线程都会对应一个响应队列，用来将响应返回给调用方