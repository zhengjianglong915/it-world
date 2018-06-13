# Redis
### 特性细节
- **内存式**： Redis将键值存储在主存中，用于快速地读写访问。
- **复制**： Redis支持主从复制。数据读取在slave完成，而数据写入在 master 完成。复制提供可伸缩性和可用性。任何一个slave宕机，其他的slave还可以提供数据访问。
- **数据结构**： Redis不仅存储字符串，还支持列表，集合，哈希和有序集合。
- **虚拟内存**： Redis使用RAM作为内存式存储。但是，在内存不足的情况下，它使用虚拟内存来保存数据。
- **发布/订阅模型**： Redis支持创建发布和订阅通道，这样Redis客户端可以订阅任意的通道来进行数据消费，并且任何已订阅该通道的客户端可以发布数据。
- **数据持久性**： Redis将内存中的数据定期保存到文件系统中。当Redis节点故障时，数据可以从Redis数据文件恢复。

### 数据类型和对应命令
有五种基本数据类型：String、Hash(哈希)、List(列表)、Set(集合)、ZSet(有序集合)。

使用不同类型保存的好处：

1. 对各个类型做不同的优化，使用不同的对象存储。
2. 对各个类型可以做校验，防止用非法方式访问对象类型。比如Set、Hash直接使用GET获取是会报错的，要求你必须按照指定类型方式去获取数据。

#### String
特点：

- string类型是**二进制安全**的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
- string类型是Redis最基本的数据类型，**一个键最大能存储512MB**。

对应的命令字： **SET**和**GET**

#### Hash（哈希）
设置使用的是 **HMSET**, 获取使用的是**HGET**. 

#### List（列表）

特点：

- Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。 
- **列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)**。


list使用的命令是 **lpush**和**lrange**.

#### Set（集合）

- 命令是**sadd**和**smembers**

#### 有序集合ZSet

命令: **zadd**和**ZRANGEBYSCORE**


## 分布式架构的5个要素
### 伸缩性
Redis通过添加节点提高集群的可用性和伸缩性。

进入集群内任意节点，使用cluster meeet命令即可使新节点加入到集群当中。新节点刚开始都是主节点状态，但是由于没有负责的槽，所以不能接受任何读写操作。对于新节点的后续操作一般有两种选择：

- 为它迁移槽和数据实现扩容。---提高伸缩性
- 作为其他主节点的从节点负责故障转移。---实现可用性

**redis-trib.rb**工具也实现了为现有集群添加新节点的命令，还实现了直接添加从节点的支持，对于上述的扩容操作，可以采用如下命令实现新节点的加入：

```
redis-trib.rb add-node 127.0.0.1:6387 127.0.0.1:6381
redis-trib.rb add-node 127.0.0.1:6388 127.0.0.1:6381
```


#### 迁移槽和数据
**reshard**是redis cluster另一核心功能，它通过迁移哈希槽来达到负载匀衡和可扩展目的。**rebalance**命令可以根据条件动态平衡各个master节点上的slots数目。

- 使用redis-trib.rb槽重分片命令
![](../images/15277346420434.jpg)
- 需要确认迁移的槽数量，输入2000个
- 输入6387的节点ID作为目标节点，目标节点只能指定一个
- 之后输入源节点的ID，这里分别输入6381、6382和6383三个节点ID最后用done表示结束
- 数据迁移之前会打印出所有的槽从源节点到目标节点的计划，确认计划无误后输入yes执行迁移工作，计划执行完成之后reshard命令自动退出。

#### reshard原理
reshard转移solt的核心部分则采用compute_reshard_table方法，compute_reshard_table方法主要用来智能计算每个目标节点需要迁移多少slot, 具体代码如下： 

```
# Given a list of source nodes return a "resharding plan"
# with what slots to move in order to move "numslots" slots to another
# instance.
def compute_reshard_table(sources,numslots)
   moved = []
   # Sort from bigger to smaller instance, for two reasons:
   # 1) If we take less slots than instances it is better to start
   #    getting from the biggest instances.
   # 2) We take one slot more from the first instance in the case of not
   #    perfect divisibility. Like we have 3 nodes and need to get 10
   #    slots, we take 4 from the first, and 3 from the rest. So the
   #    biggest is always the first.
   // 1) 通过slot个数对源节点进行排序，slot多的排在前面
   sources = sources.sort{|a,b| b.slots.length <=> a.slots.length}
   // 2) 计算源节点的slot总个数
   source_tot_slots = sources.inject(0) {|sum,source|
       sum+source.slots.length
   }
   // 3) 可以看到按照节点占slot总数的百分比来迁移slot，及slot个数越多的节点将被迁移更多。
   // 还可以看到slot节点最多的节点会为slot的最大整数
   sources.each_with_index{|s,i|
       # Every node will provide a number of slots proportional to the
       # slots it has assigned.
       n = (numslots.to_f/source_tot_slots*s.slots.length)
       if i == 0
           n = n.ceil
       else
           n = n.floor
       end
       // 4) 将slot的分派到节点的信息插入moved变量中
       s.slots.keys.sort[(0...n)].each{|slot|
           if moved.length < numslots
               moved << {:source => s, :slot => slot}
           end
       }
   }
   return moved
end

def show_reshard_table(table)
   table.each{|e|
       puts "    Moving slot #{e[:slot]} from #{e[:source].info[:name]}"
   }
end
```

执行compute_reshard_table方法，计算需要迁移的slot数量如何分配到源节点列表，采用的算法是按照节点负责slot数量由多到少排序，计算每个节点需要迁移的slot的方法为：迁移slot数量 * (该源节点负责的slot数量 / 源节点列表负责的slot总数)。具体步骤如下：

- 通过slot个数对源节点进行排序，slot多的排在前面。
- 计算源节点的slot总个数
- 可以看到按照节点占slot总数的百分比来迁移slot，及slot个数越多的节点将被迁移更多。还可以看到slot节点最多的节点会为slot的最大整数。
- 将slot的分派到节点的信息插入moved变量中



## 分布式锁实现原理
Redis为单进程单线程模式，采用队列模式将并发访问变成串行访问，且多客户端对Redis的连接并不存在竞争关系。

### 单实例实现 
利用的是 redis锁定的原理是利用setnx命令，即只有在某个key不存在情况才能set成功该key，这样就达到了多个进程并发去set同一个key，只有一个进程能set成功。

Redis 2.6.12 版本开始，redis的set命令直接直接设置NX和EX属性，NX即附带了setnx数据，key存在就无法插入，EX是过期属性，可以设置过期时间。这样一个命令就能原子的完成加锁和设置过期时间。命令格式如下：

```
SET resource_name my_random_value NX PX 30000
```

**这个key的值设为“my_random_value”。这个值必须在所有获取锁请求的客户端里保持唯一**。 基本上这个随机值就是用来保证能安全地释放锁。

这个很重要，因为这可以避免误删其他客户端得到的锁，举个例子，一个客户端拿到了锁，被某个操作阻塞了很长时间，过了超时时间后自动释放了这个锁，然后这个客户端之后又尝试删除这个其实已经被其他客户端拿到的锁。所以单纯的用DEL指令有可能造成一个客户端删除了其他客户端的锁，用上面这个脚本可以保证每个客户单都用一个随机字符串’签名’了，这样每个锁就只能被获得锁的客户端删除了。

```
if redis.call("get",KEYS[1]) == ARGV[1] then
        return redis.call("del",KEYS[1])
    else
        return 0
    end
```

这个随机字符串应该用什么生成呢？我假设这是从/dev/urandom生成的20字节大小的字符串，但是其实你可以有效率更高的方案来保证这个字符串足够唯一。比如你可以用RC4加密算法来从/dev/urandom生成一个伪随机流。还有更简单的方案，比如用毫秒的unix时间戳加上客户端id，这个也许不够安全，但是也许在大多数环境下已经够用了。

**key值的超时时间，也叫做”锁有效时间”。这个是锁的自动释放时间，也是一个客户端在其他客户端能抢占锁之前可以执行任务的时间，这个时间从获取锁的时间点开始计算。**

缓存锁优势是性能出色，劣势就是由于数据在内存中，一旦缓存服务宕机，锁数据就丢失了。像redis自带复制功能，可以对数据可靠性有一定的保证，但是由于复制也是异步完成的，因此依然可能出现master节点写入锁数据而未同步到slave节点的时候宕机，锁数据丢失问题。

### 分布式环境---Redlock算法

在分布式版本的算法里我们假设我们有N个Redis master节点，这些节点都是完全独立的，我们不用任何复制或者其他隐含的分布式协调算法。我们已经描述了如何在单节点环境下安全地获取和释放锁。因此我们理所当然地应当用这个方法在每个单节点里来获取和释放锁。在我们的例子里面我们把N设成5，这个数字是一个相对比较合理的数值，因此我们需要在不同的计算机或者虚拟机上运行5个master节点来保证他们大多数情况下都不会同时宕机。一个客户端需要做如下操作来获取锁：

- 获取当前时间（单位是毫秒）。
- 轮流用相同的key和随机值在N个节点上请求锁，在这一步里，客户端在每个master上请求锁时，会有一个和总的锁释放时间相比小的多的超时时间。比如如果锁自动释放时间是10秒钟，那每个节点锁请求的超时时间可能是5-50毫秒的范围，这个可以防止一个客户端在某个宕掉的master节点上阻塞过长时间，如果一个master节点不可用了，我们应该尽快尝试下一个master节点。
- 客户端计算第二步中获取锁所花的时间，只有当客户端在大多数master节点上成功获取了锁（在这里是3个），而且总共消耗的时间不超过锁释放时间，这个锁就认为是获取成功了。
- 如果锁获取成功了，那现在锁自动释放时间就是最初的锁释放时间减去之前获取锁所消耗的时间。
- 如果锁获取失败了，不管是因为获取成功的锁不超过一半（N/2+1)还是因为总消耗时间超过了锁释放时间，客户端都会到每个master节点上释放锁，即便是那些他认为没有获取成功的锁。


#### 失败的重试
当一个客户端获取锁失败时，这个客户端应该在一个**随机延时后**进行重试，之所以采用随机延时是为了避免不同客户端同时重试导致谁都无法拿到锁的情况出现。同样的道理客户端越快尝试在大多数Redis节点获取锁，出现多个客户端同时竞争锁和重试的时间窗口越小，可能性就越低，所以最完美的情况下，客户端应该用多路传输的方式同时向所有Redis节点发送SET命令。这里非常有必要强调一下客户端如果没有在多数节点获取到锁，一定要尽快在获取锁成功的节点上释放锁，这样就没必要等到key超时后才能重新获取这个锁（但是如果网络分区的情况发生而且客户端无法连接到Redis节点时，会损失等待key超时这段时间的系统可用性）。

#### 存在以下特点

- 首先必须部署5个节点才能让Redlock的可靠性更强。
- 然后需要请求5个节点才能获取到锁，通过Future的方式，先并发向5个节点请求，再一起获得响应结果，能缩短响应时间，不过还是比单节点redis锁要耗费更多时间。
- 然后由于必须获取到**5个节点中的3个以上**，所以可能出现获取锁冲突，即大家都获得了1-2把锁，结果谁也不能获取到锁，这个问题，redis作者借鉴了**raft算法**的精髓，通过冲突后在**随机时间开始**，可以大大降低冲突时间，但是这问题并不能很好的避免，特别是在第一次获取锁的时候，所以获取锁的时间成本增加了。
- 如果5个节点有2个宕机，此时锁的可用性会极大降低，首先必须等待这两个宕机节点的结果超时才能返回，另外只有3个节点，客户端必须获取到这全部3个节点的锁才能拥有锁，难度也加大了。
- 如果出现网络分区，那么可能出现客户端永远也无法获取锁的情况。

参考： 

- [《Redis官方文档》用Redis构建分布式锁](http://ifeve.com/redis-lock/)
- [聊一聊分布式锁的设计](http://weizijun.cn/2016/03/17/聊一聊分布式锁的设计/)

### 更好的分布式锁--zookeeper
zookeeper还有几个特质，让它非常适合作为分布式锁服务。

- zookeeper支持watcher机制，这样实现阻塞锁，可以watch锁数据，等到数据被删除，zookeeper会通知客户端去重新竞争锁。
- zookeeper的数据可以支持临时节点的概念，即客户端写入的数据是临时数据，在客户端宕机后，临时数据会被删除，这样就实现了锁的异常释放。使用这样的方式，就不需要给锁增加超时自动释放的特性了。

## 参考

- [Redis集群伸缩](https://www.cnblogs.com/skyer5217/p/8994390.html)
- [redis cluster源码研究--reshard](https://blog.csdn.net/whycold/article/details/42585967)
- [Redis 学习笔记（十五）Redis Cluster 集群扩容与收缩](https://blog.csdn.net/men_wen/article/details/72896682)
- [redis cluster管理工具redis-trib.rb详解](http://weizijun.cn/2016/01/08/redis%20cluster%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7redis-trib-rb%E8%AF%A6%E8%A7%A3/)


