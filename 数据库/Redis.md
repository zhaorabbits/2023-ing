# Redis

[TOC]

## 基础概念





## 面试题

1. 简单介绍一下Redis？

   Redis就是一个使用C语言开发的数据库，不过与传统数据库不同的是Redis的数据是存在内存中的，也就是它是内存数据库，所以读写速度非常快，因此Redis被广泛应用于缓存方向。

   另外，Redis除了做缓存之外，Redis也经常用来做分布式锁，甚至是消息队列。

   Redis提供了多种数据类型来支持不同的业务场景。Redis还支持事务、持久化、Lua脚本、多种集群方案。

2.  分布式缓存常见的技术选型方案有那些？

    分布式缓存的话，使用的比较多的主要是Memcached和Redis。不过，现在基本没有看过还有项目使用Memcached来做缓存，都是直接用Redis。

    Memcached是分布式缓存最开始兴起的那会，比较常用的。后来，随着Redis的发展，大家慢慢都转而使用更加强大的Redis了。

    分布式缓存主要解决的是单机

3.  说一下Redis和Memcached的区别和共同点？

    共同点：

    - 都是基于内存的数据库，一般都用来当作缓存使用
    - 都有过期策略
    - 两者的性能都非常高

    区别：

    - Redis支持更丰富的数据类型（支持更复杂的应用场景）。Redis不仅仅支持简单的k/v类型的数据，同时还提供阿里list、set、zset、hash等数据结构的存储。Memcached只支持最简单的k/v数据类型。
    - Redis支持数据持久化，可以把内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用，而Memcache把数据全部存在内存之中。
    - Redis有灾难恢复机制。因为可以把缓存中的数据持久化到磁盘上。
    - Redis在服务器内存使用完之后，可以将不用的数据放到磁盘上。但是，Memcached在服务器内存使用完之后，就会直接报异常。
    - Memcached没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是Rredis目前是原生支持cluster模式的。
    - Memcached是多线程，非阻塞IO复用的网络模型；Redis使用单线程的多路IO服用模型
    - Redis支持发布订阅模型、Lua脚本、事务等功能，而Memecached不支持。并且，Redis支持更多的编程语言
    - Memcached过期数据的删除策略只用了惰性删除，而Redis同时使用了惰性删除与定期删除。

4. redis数据库中的数据结构 ? 底层用了哪些数据结构 ?

   https://blog.csdn.net/a745233700/article/details/113449889   Redis object

   - string：

     - string数据结构是简单的key-value类型。虽然Redis是用C语言写的，但是Redis并没有使用C的字符串表示，而是自己构建了一种简单动态字符串（simple dynamic string，SDS）。相比于C的原生字符串，Redis的SDS不光可以保存文本数据还可以保存二进制数据，并且获取字符串长度复杂度为O（1）（C字符串为O（N））,除此之外，Redis的SDS API是安全的，不会造成缓冲区溢出。

     - 常用命令：

       ```
       set key value #设置key-value
       get key
       exists key #判断某个key是否存在
       strlen key #返回key所存储的字符串值的长度
       del key # 删除某个key对应的值
       
       mset key1 value1 key2 value2 # 批量设置
       mget key1 key2
       
       set number 1
       incr number#将value数字值加1
       decr number #将数字值减1
       
       expire key 60 #数据在60s 后过期
       ttl key #查看数据还有多久过期
       ```

       

     - 应用场景：一般常用在需要技术的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

     - 底层实现：SDS：动态字符串

       - 可以动态扩展内存
       - 采用预分配冗余空间的方式来减少内存的频繁分配，从而优化字符串的增长操作。

   - list：

     - 链表。Redis实现了自己的链表数据结构，Redis的list实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

     - 常用命令：

       ```
       rpush muList value1 #向list头部（右边）添加元素
       
       rpush myList value2 value3 
       lpop myList  #将list的尾部（左边）元素取出
       lrange myList 0 1 # 查看对应下标的list列表，0为start， 1为end   闭区间
       lrange myList 0 -1 # 查看列表中的所有元素
       lrange命令，可以基于list实现分页查询，性能非常高！
       
       llen myList  #查看链表长度
       ```

       

     - 发布与订阅或者说消息队列、慢查询。

     - 底层实现：

       - Redis3.2之前：压缩列表ziplist或者双向循环链表linkedlist
       - Redis3.2之后：quicklist（基于ziplist）
         - quicklist：ziplist（不易修改）和linked（有内存碎片，不易随机存取）折中
           - quicklist是一个基于ziplist的双向链表，quicklist的每个节点都是一个ziplist

   - hash：

     - hah类似于JDK1.8前的HashMap，内部实现也差不多（数组+链表）。不过。Redis的hash做了更多优化。另外，hash是一个string类型的field和value映射表，特别适合用于存储对象，后续操作的时候，可以直接仅仅修改这个对象中的某个字段的值。比如我们可以hash数据结构来存储用户信息，商品信息等等
     
     - 常用命令：
     
       ```
       hset userInfoKey name "guide" description "dev" age "24"
       hexists userInfoKey name # 查看key对应的value中指定字段是否存在
       hget userInfoKey name #获取存储在哈希表中指定字段的值
       hget userInfoKey age
       hgetall userInfoKey #获取在哈希表中指定key的所有字段和值
       
       hkeys userInfoKey # 获取key列表
       hvals userInfoKey # 获取value列表
       
       hset userInfoKey name “GuideGeGe”
       ```
     
       
     
     - 应用场景：系统中对象数据的存储
     
     - 底层实现：压缩列表ziplist或者字典dict
     
       - dict：
         - 是一个用于维护key-value映射关系的数据结构
         - 本质上是为了解决查找问题，是一个基于哈希表的算法。采用某个哈希函数并通过计算key从而找到在哈希表中的位置，采用拉链法解决冲突，并在装载因子超过预定值时自动扩展内存，引发重哈希（rehashing），为了避免扩容时一次性对所有key进行重哈希，Redis采用了一种增量式重哈希的方法，将重哈希的操作分散到对于dict的各个增删改查的操作中区。这种方法能够做到每次只对一小部分key进行重哈希，而每次重哈希之间不影响dict操作。避免重哈希期间单个请求的的响应时间剧烈增加，这与“快速响应时间”的涉及原则是相符的。
         - 当装载因子大于1的时候，Redis会触发扩容，将散列表扩大为原来大小的2倍左右；当数据动态减少之后，为了节省内存，当装载因子小于0.1的时候，Redis就会触发缩容，缩小为字典中数据个数的大约2倍左右。
       - ziplist：
         - 使用一块连续的内存空间来存储数据，并采用可变长的编码方式，支持不同类型和大小的数据的存储，更加节省内存，而且数据存储在一片连续的内存空间，读取的效率也非常高。
     
   - set：

     - set类似于Java中的HashSet。Redis中的set类型是一种无序集合，集合中的元素没有先后顺序。当你需要存储一个列表数据，又不希望出现重复数据是，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的，可以基于set轻易实现交集、并集、差集的操作。比如：你可以将一个用户所有关注人存在一个集合中，将其所有粉丝存在一个集合。Redis可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。

     - 常用命令：

       ```
       sadd mySet value1 value2 #添加元素进去
       sadd mySet value1 # 不允许有重复元素
       smembers mySet # 查看set中所有的元素
       scard mySet #查看set 的长度
       
       sismember mySet value1 # 检查某个元素是否存在在set中，只能接收单个元素
       sadd mySet2 value2 value3 
       sinterstore mySet3 mySet mySet2 # 获取mySet 和mySet2 的交集并存放在mySet3中
       smembers mySet3
       ```

       

     - 应用场景：需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景。

     - 底层实现：有序整数集合intset或者字典dict

       - intset：
         - 由整数组成的有序集合，从而便于进行二分查找，用于快速判断一个元素是否属于这个集合。它在内存分配上于ziplist有些类似，是一个连续的整块内存空间，而且队员大整数和小整数（按绝对值）采用不同的编码，尽量对内存的使用进行了优化。
         - intset与ziplist相比：
           - ziplist可以存储任意二进制串，而intset只能存储整数
           - ziplist是无序的，而intset是从小到大有序的，查找性能更高
           - ziplist可以对每个数据项进行不同的变长编码（每个数据项前面都有数据长度字段len），而intset只能整体使用一个统一的编码。

   - sorted set：

     - 和set相比，sorted set增加了一个权重参数score，使得集合中的元素能够按score进行有序排列，还可以通过score的范围来获取元素的别表。有点像是Java中HashMap和TreeSet的结合体。

     - 常用命令：

       ```
       zadd myZset 3.0 value1 #添加元素sorted set中3.0为权重
       
       zadd myZset 2.0 value2 1.0 value3  #一次添加多个元素
       zcard myZset #查看sorted set 中的元素数量
       zscore myZset value1 #查看某个value的权重
       
       zrange myZset 0 -1 # 顺序输出某个范围区间的元素，0 -1 表示输出所有元素
       zrange myZset 0 1  # 顺序输出某个范围区间的元素，0 为start 1为stop
       zrevrange myZset 0 1  # 逆序输出某个范围区间的元素，0为start 1为stop
       ```

       

     - 应用场景：需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的信息排行榜）等信息。
     - 底层实现：压缩列表ziplist或者zset。zset（dict+skiplist）
       - skiplist：跳表 （平均时间复杂度O(logn)）
         - 

5. 缓存数据的处理流程是怎么样的？

   1. 如果用户请求的数据在缓存中就直接返回。
   2. 缓存中不存在的话就看数据库中是否存在。
   3. 数据库中存在的话就更新缓存中的数据。
   4. 数据库中不存在的话就返回空数据。

6. 为什么要用缓存（Redis）？

   - 高性能：直接操作内存速度更快
   - 高并发：
     - 一般向MySQL这类的数据库的QPS（Query Per Second：服务器每秒可以执行的查询次数）大概在1w左右（4核8g），但是使用Redis缓存之后，很容易达到10w+，甚至最高达到30W+，（就单机Redis的情况，Redis集群的话会更高）
     - 直接操作缓存能够承受的数据库请求数量是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。进而，我们也就提高了系统整体的并发。

7. 你项目里用到了redis，说下你怎么用redis的，redis还有哪些具体应用场景?

8. redis的持久化有哪些方式?

9. redis是单线程，为什么性能还这么好，说具体一点 ?

10. redis宕机怎么办? 

11. 谈谈 Redis 如果出现不一致是什么原因，解决思路？

12. Redis大Key，会引发哪些问题？

13. 怎么解决大Key问题？拆

14. Redis怎么实现分布式锁？

