# Redis
   Redis是一个开源的key-value存储系统。

   和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set –有序集合)和hash（哈希类型）。

   这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。

   在此基础上，Redis支持各种不同方式的排序。

   与memcached一样，为了保证效率，数据都是缓存在内存中。

   区别的是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。

   并且在此基础上实现了master-slave(主从)同步。

   Redeis 是单线程 + 多路IO复用技术。默认16个数据库，默认用0号库。

   memcached：多线程+锁.
#### 应用场景
###### 配合关系型数据库做高速缓存
   高频次，热门访问的数据，降低数据库IO。
   
   分布式架构，做session共享。
###### 多样的数据结构存储持久化数据
​
## Redis 键（Key）
    keys * 查看当前库中所有key
    exists key 判断key是否存在
    type key 查看key类型
    del key 删除指定key数据
    unlink key （异步删除）根据value选择非阻塞删除，仅将key从keyspace元数据中删除，真正的删除会在后续异步操作。
    expire key 10 10秒钟：为给定的key设置过期时间
    ttl key 查看key还有多少秒过期，-1表示永不过期，-2表示已过期
    select 命令切换数据库
    dbsize 查看当前库的key的数量
    flushdb 清空当前库
    flushall 通杀全部库

## 一. Redis 字符串（String）
    最基本的类型
    二进制安全的，最基本的数据类型
    一个Redis中字符串value最多可以是512M。

### 常用命令
    set key value 设置，设置相同key会覆盖value
    get key 取值
    append key value 将给定的value追加到原值的末尾
    strlen key 获得值的长度
    setnx key value 只有在key不存在时，设置key的值
    incr key 将key中储存的数字值增1，只能对数字操作，如果为空，新增值为1
    decr key 将key中储存的数字值减1，只能对数字操作，如果为空，新增值为-1
    incrby/decrby key 长度 : 将key 中储存的数字值增减，自定义长度

    原子性。所谓原子操作是指不会被线程调度机制打断的操作。
    这种操作一旦开始，就一直运行到结束，中间不会有任何context switch（切换到另一个线程）。
    （1）在单线程中，能够在单条指令中完成的操作都可以认为时“原子操作”，因为中断只能发生在指令之间。
    （2）在多线程中，不能被其它进程（线程）打断的操作就叫原子操作。
     Redis单命令的原子性主要得益于Redis的单线程。

    mset key value key1 value1 同时设置一个或多个key-value
    mget key key1 同时获取一个或多个value
    msetnx key value key1 value1 同时设置一个或多个key-value,当且仅当给定key都不存在
    原子性，有一个失败都失败

### 数据结构
   String的数据结构为简单动态字符串。

## 二. Redis列表(List)
    单键多值
    Redis列表简单的字符串类型，按照插入顺序排序。
    
   它的底层实际是个<font color="red">双向列表</font>，对两端的操作性能很高，通过索引下标的操作中间的的节点性能会较差。

### 常用命令
    lpush/rpush key value1 value2... 从左边/右边插入一个或多个值
    lpop/rpop key 从左边/右边吐出一个值(取出值)。值在键在，值光键亡。
    rpoplpush key1 key2 从key1列表右边吐出一个值，插到key2列表左边。
    lrange key start stop 按照索引下标获得元素（从左到右）
        0 -1 0左边第一个，-1右边第一个，0-1表示获取所有
    lindex key index 按照索引下标获得元素(从左到右)
    llen key 获得列表长度
    linsert key before value newvalue 在value的后面插入newvalue
    lrem key n value 从左边删除n个value（从左到右）
    lset key index value 将列表key下标为index的值替换成value

### 数据结构
  List的数据结构为快速链表quickList。

  首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也既是压缩列表。

  它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

  当数据量两比较多的时候才会改成quicklist。

  因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

  Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

## 三. Redis集合（Set）
  set 对外提供的功能与list类似，是一个列表的功能，特殊之处在于set是可以<font color="red">自动排重</font>的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。
  
  Redis的Set是String 类型的<font color="red">无序集合。它底层其实是一个value为null的hash表</font>，所以添加、删除、查找的<font color="red">**复杂度都是O(1)**</font>。 

### 常用命令
    sadd key value1 value2 将一个或多个member元素加入到集合key中，已经存在的member元素将被忽略
    smembers key 取出该集合的所有值
    sismember key value 判断集合key是否为含有该value值，有1，没有0
    scard key 返回该集合的元素个数
    srem key value1 value2 ... 删除结合中的某个元素
    spop key 随机从该集合中吐出一个值
    srandmamber key n 随机从该集合中取出n个值，不会从集合中删除
    smove source destination value 把集合中一个值从一个集合移动到另一个集合
    sinter key1 key2 返回两个集合的交集元素
    sunion key1 key2 返回两个集合的并集元素
    sdiff key1 key2 返回两个集合的差集元素(key1中的，不包含key2中的)

### 数据结构
   Set数据结构是dict字典，字典是用哈希表实现的。


## 四. Redis 哈希(Hash)
    Redis hash 是一个键值对集合。
    Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。类似Java里面的Map<String,Object>。   

### 常用命令
    hset key field value 给key集合中的field 键赋值value
    hget key1 field 从key1集合field取出value
    hmset key1 field1 value1 field2 value2 批量设置hash的值
    hexists key1 field 查看哈希表 key 中，给定域field是否存在
    hkeys key 列出该hash集合的所有field
    hvals key 列出该hash集合的所有value
    hincrby key field increment 为哈希表key中的域field的值加上增量 1 -1
    hsetnx key field value 将哈希表key中的域field 的值设置为value，当且仅当域field不存在.

### 数据结构
   Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

## 五. Redis 有序集合(Zset)
   Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的集合。
   不同之处是有序集合的每个成员都关联了一个<font color="red">评分(score)</font>,这个评分(score)被用来按照从最低分到最高分的方式排序集合中的成员。<font color="red">集合的成员是唯一的，但是评分可以是重复的</font>。
   
   因为元素是有序的，所以你也可以很快的根据评分(score)或者次序(position)来获取一个范围的元素。

   访问有序集合的中间元素也是非常快的，因此你能够使用有序集合作为一个没有重复成员的智能列表。

### 常用命令
    zadd key score1 value1 score2 value2 将一个或多个member元素及其score值加入到有序集合key当中。
    zrange key start stop WITHSCORES 返回有序集合key中，下标在start stop之间的元素，带WITHSCORES，可以让分数一起和值返回到结果。
    zrangebyscore key minmax withscores limit offset count 返回有序集合key中，所有score值介于min和max之间(包括等于min或max)的成员。有序集合成员按score值递增(从小到大)次序排列。
    zrevrangebyscore key minmax withscores limit offset count 同上，改为从大到小排列。
    zincrby key increment value 为元素的score加上增量
    zrem key value 删除该集合下，指定值的元素
    zcount key min max 统计该集合，分数区间内的元素个数
    zrank key value 返回该集合中的排名，从0开始。

### 数据结构
   SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

   zset底层使用了两个数据结构：
   (1) hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

   (2) 跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。
