Redis
<!-- TOC -->

- [Redis 官网](#redis-官网)
- [Redis 概况](#redis-概况)
    - [Redis 是什么？](#redis-是什么)
    - [Redis 数据结构](#redis-数据结构)
        - [value 对应的五种数据结构](#value-对应的五种数据结构)
        - [Redis 核心对象 redisObject](#redis-核心对象-redisobject)
            - [编码方式（encoding）](#编码方式encoding)
        - [Redis 五种数据结构对应的内部编码](#redis-五种数据结构对应的内部编码)
    - [reference](#reference)
- [搭建 Redis 环境](#搭建-redis-环境)
    - [启动 redis server](#启动-redis-server)
    - [启动 redis-cli](#启动-redis-cli)
    - [redis-cli连接远程服务](#redis-cli连接远程服务)
    - [reference](#reference-1)
- [Redis命令](#redis命令)
    - [Redis keys 命令](#redis-keys-命令)
    - [Redis 字符串命令](#redis-字符串命令)
    - [Redis hash 命令](#redis-hash-命令)
    - [Redis list 命令](#redis-list-命令)
    - [Redis set 命令](#redis-set-命令)
    - [Redis sorted set 命令](#redis-sorted-set-命令)
    - [Redis HyperLogLog 命令](#redis-hyperloglog-命令)
    - [reference](#reference-2)
- [Redis 持久化](#redis-持久化)
    - [RDB](#rdb)
    - [AOF](#aof)
    - [持久化最佳策略](#持久化最佳策略)
    - [reference](#reference-3)
- [Redis 实现分布式锁](#redis-实现分布式锁)
    - [单机模式的Redis分布式锁](#单机模式的redis分布式锁)
    - [集群模式的Redis分布式锁 Redlock](#集群模式的redis分布式锁-redlock)

<!-- /TOC -->
### Redis 官网
Redis is an in-memory database that persists on disk. The data model is key-value, but many different kind of values are supported: Strings, Lists, Sets, Sorted Sets, Hashes, HyperLogLogs, Bitmaps. 
- http://redis.io
- http://www.redis.cn
- https://github.com/antirez/redis
### Redis 概况

#### Redis 是什么？
Redis是一个开源（BSD许可）的，利用内存进行存储的数据结构存储系统；它可以用作数据库、缓存和消息中间件。
- redis由意大利人 Salvatore Sanfilippo 使用C语言开发
- redis支持字符串（string）、列表（list）、集合（set）、有序集合（zset）、散列表（hash）五种基本数据结构类型
- redis从 2.2.0 版本开始支持bitmap；在 2.8.9 版本添加了 HyperLogLog 用以进行基数统计；在 3.2 版本中新增了对GEO(地理位置)的支持
- redis支持简单事物与数据持久化，提供 RDB、AOF两种可选的持久化方式
- redis可以用作数据库、缓存、消息队列等

#### Redis 数据结构
##### value 对应的五种数据结构
Redis存储key-value键值对数据，其中key类型为字符串，value对应五种数据结构，如下图所示：

![value对应的五种数据结构](/imgs/redis/5_data_structure.PNG)
- 字符串（string）类型的数据结构，对应的就是一个普通的字符串
- 散列表（hash）类型的数据结构，对应的就是一个hash table，散列表特别适合用于存储对象
- 列表（list）类型的数据结构，对应的就是一个双向列表，按照插入顺序排序
- 集合（set）类型的数据结构，对应的就是一个string类型的无序集合，集合中的数据不能重复出现
- 有序集合（zset）类型的数据结构, 对应的就是一个string类型的有序集合，排序因子为每个元素附带的一个double型的分数

##### Redis 核心对象 redisObject
在redis的 key-value存储系统中，value 类型则为 redis 对象 redisObject, redisObject对象可以绑定对应的五种数据类型，如下图所示：

![redisObject](/imgs/redis/redisObject.PNG)
- 数据类型(type)，对应五种数据类型
- 编码方式(encoding)，指定所绑定数据类型的编码方式
- 数据指针(ptr), 指向对象底层实现的数据结构
- 虚拟内存(vm), 该功能默认处于关闭状态，只有打开了redis的虚拟内存功能，才会给vm分配真正的内存

###### 编码方式（encoding）
- raw RAW编码方式使用简单动态字符串来保存字符串对象,才有预分配空间的方式来避免字符串修改时频繁的分配释放内存
- int INT编码方式以整数保存字符串数据，仅限能用long类型值表达的字符串
- embstr 从Redis 3.0版本开始字符串引入了EMBSTR编码方式，长度小于OBJ_ENCODING_EMBSTR_SIZE_LIMIT(39)的字符串将以EMBSTR方式存储。采用这个方式可以减少内存分配的次数，提高内存分配的效率，降低内存碎片率。
- hashtable 当数据类型无法满足使用ziplist的条件时，Redis会使用hashtable作为数据结构的内部实现
- ziplist 列表(List),散列表(Hash),有序集合(Sorted Set)在成员较少，成员值较小的时候都会采用压缩列表(ZIPLIST)编码方式进行存储；成员值"较小"的标准可以通过配置项进行配置；压缩列表简单来说就是一系列连续的内存数据块，其内存利用率很高，但增删改查效率较低，所以只会在成员较少，值较小的情况下使用。
- linkedlist 在Redis 3.2版本之前，一般的链表使用LINKDEDLIST编码。在Redis 3.2版本开始，所有的链表都是用QUICKLIST编码。两者都是使用基本的双端链表数据结构，区别是QUICKLIST每个节点的值都是使用ZIPLIST进行存储的。
- skiplist 跳跃表(SKIPLIST)编码方式为有序集合对象专用，有序集合对象采用了字典+跳跃表的方式实现；其中字典里面保存了有序集合中member与score的键值对，跳跃表则用于实现按score排序的功能
- intset 当一个集合只包含整数值元素， 并且这个集合的元素数量不多时， Redis 就会使用整数集合作为集合键的底层实现

Redis这种通过redisObject指定数据结构编码方式的设计有两个好处：
- 可以改进内部编码，而对外的数据结构和命令没有影响，这样一旦开发开发出优秀的内部编码，无需改动外部数据结构和命令。
- 多种内部编码实现可以在不同场景下发挥各自的优势。例如ziplist比较节省内存，但是在列表元素比较多的情况下，性能会有所下降，这时候Redis会根据配置选项将列表类型的内部实现转换为linkedlist。

##### Redis 五种数据结构对应的内部编码
Redis在不同的情况下会为数据对象选择适合的编码方式

![Redis 五种数据结构对应的内部编码](/imgs/redis/5_data_encoding.PNG)
- string的内部编码有三种 
    - int：8个字节的长整型
    - embstr：小于等于39个字节的字符串
    - raw：大于39个字节的字符串
- hash 
    - ziplist（压缩列表）：当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个），同时所有值都小于hash-max-ziplist-value配置（默认64个字节）时，Redis会使用ziplist作为哈希的内部实现ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀
    - hashtable（哈希表）：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现。因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O(1)
- list
    - ziplist（压缩列表）：当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个）同时所有值都小于hash-max-ziplist-value配置（默认64个字节）时，Redis会使用ziplist作为哈希的内部实现
    - linkedlist（链表）：当列表类型无法满足ziplist的条件时，Redis会使用linkedlist作为列表的内部实现
- set
    - intset（整数集合）：当集合中的元素都是整数且元素个数小于set-max-intset-entries配置（默认512个）时，Redis会选用intset来作为集合内部实现，从而减少内存的使用。
    - hashtable（哈希表）：当集合类型无法满足intset的条件时，Redis会使用hashtable作为集合的内部实现
- zset
    - ziplist（压缩列表）：当有序集合的元素个数小于zset-max-ziplist-entries配置（默认128个）同时每个元素的值小于zset-max-ziplist-value配置（默认64个字节）时，Redis会用ziplist来作为有序集合的内部实现，ziplist可以有效减少内存使用
    - skiplist（跳跃表）：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时zip的读写效率会下降



#### reference
- [redis中文官网](http://www.redis.cn/)
- [菜鸟教程-Redis](http://www.runoob.com/redis/redis-tutorial.html)
- [Redis数据编码方式详解](https://yq.aliyun.com/articles/63461)
- [Redis的五种数据结构的内部编码](https://www.cnblogs.com/yangmingxianshen/p/8054094.html)


### 搭建 Redis 环境

由于Redis对windows的支持不友好，所以这儿介绍使用docker容器来启动 redis(只用于体验redis,不涉及各种详细配置)
#### 启动 redis server
- 拉取 redis 镜像
```shell
# 拉取 redis 镜像，不输入version时，默认拉取最新的发行版
# 命令 docker pull redis:[version]
λ  docker pull redis
λ  docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
redis                          latest              c5355f8853e4        3 months ago        107MB
```
- 启动容器
```shell
# 从 redis 镜像启动一个容器，命名为 redis-S
# -d 后台运行
# -name 指定容器的名称
# -v 给redis的data目录挂载本地磁盘空间($PWD/data 当前目录下的data)
λ  docker run -d --name redis-S -v $PWD/data:/data redis redis-server
λ  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
bed6a2a9b3bc        redis               "docker-entrypoint.s…"   6 weeks ago         Up 25 hours         6379/tcp            redis-S
```
#### 启动 redis-cli

- 直接启动 redis-S 容器的 redis-cli
```shell
# 执行如下命令，进入客户端
λ  docker exec -it bed6 redis-cli
127.0.0.1:6379> keys *
1) "xxx"
2) "testzset"
127.0.0.1:6379>
```
- 启动一个新容器链接到 redis-S, 开启 redis-cli 客户端
```shell
# -it是交互模式(-i: 以交互模式运行容器,-t: 为容器重新分配一个伪输入终端) 
# –link 连接另一个容器,这样就可以使用容器名作为host了 
# –rm 在容器退出时就能够自动清理容器内部的文件系统, --rm选项也会清理容器的匿名data volumes, 执行docker run命令带--rm命令选项，等价于在容器退出后，执行docker rm -v
λ  docker run -it --link redis-S --rm redis redis-cli -h redis-S -p 6379
redis-S:6379> keys *
1) "xxx"
2) "testzset"
redis-S:6379>
```
#### redis-cli连接远程服务
用法：redis-cli [OPTIONS] [cmd [arg [arg ...]]]

-h <主机ip>，默认是127.0.0.1

-p <端口>，默认是6379

-a <密码>，如果redis加锁，需要传递密码

--help，显示帮助信息
```shell
haiyoung@haiyoung ~ $ redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> ping
PONG
```

#### reference
- [Docker 安装 Redis](http://www.runoob.com/docker/docker-install-redis.html)
- [Docker 容器启动 redis](https://www.yuque.com/haiyoung/useful_notes/rpb8zg)

### Redis命令
#### Redis keys 命令
<style>table th:first-of-type { width: 30px;}</style>
<style>table th:nth-of-type(2) { width: 200px;}</style>
|序号|命令|描述|
|:------|:------|:------|
|1|DEL key|该命令用于在 key 存在时删除 key，返回被删除 key 的数量|
|2|DUMP key|序列化给定 key ，并返回被序列化的值|
|3|EXISTS key|检查给定 key 是否存在，若 key 存在返回 1 ，否则返回 0 |
|4|EXPIRE key seconds|为给定 key 设置过期时间，设置成功返回 1 |
|5|EXPIREAT key timestamp|EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)，设置成功返回 1|
|6|PEXPIRE key milliseconds|设置 key 的过期时间以毫秒计，设置成功，返回 1,key 不存在或设置失败，返回 0|
|7|PEXPIREAT key milliseconds-timestamp|设置 key 过期时间的时间戳(unix timestamp) 以毫秒计, 设置成功返回 1|
|8|	KEYS pattern|查找所有符合给定模式( pattern)的 key,返回符合给定模式的 key 列表 (Array)|
|9|MOVE key db|将当前数据库的 key 移动到给定的数据库 db 当中,移动成功返回 1 ，失败则返回 0|
|10|PERSIST key|移除 key 的过期时间，key 将持久保持，当过期时间移除成功时，返回 1 。 如果 key 不存在或 key 没有设置过期时间，返回 0|
|11|PTTL key|以毫秒为单位返回 key 的剩余的过期时间,当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间|
|12|TTL key|以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live),当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间|
|13|RANDOMKEY|从当前数据库中随机返回一个 key，当数据库不为空时，返回一个 key 。 当数据库为空时，返回 nil （windows 系统返回 null）|
|14|RENAME key newkey|修改 key 的名称，改名成功时提示 OK ，失败时候返回一个错误。当 OLD_KEY_NAME 和 NEW_KEY_NAME 相同，或者 OLD_KEY_NAME 不存在时，返回一个错误。 当 NEW_KEY_NAME 已经存在时， RENAME 命令将覆盖旧值|
|15|RENAMENX key newkey|仅当 newkey 不存在时，将 key 改名为 newkey，修改成功时，返回 1 。 如果 NEW_KEY_NAME 已经存在，返回 0 |
|16|TYPE key|返回 key 所储存的值的类型|

#### Redis 字符串命令
- SET key value 设置指定 key 的值
- GET key 获取指定 key 的值
- GETRANGE key start end 返回 key 中字符串值的子字符
- GETSET key value 将给定 key 的值设为 value ，并返回 key 的旧值(old value)
    ```java
    127.0.0.1:6379> keys *
    (empty list or set)
    127.0.0.1:6379> set key001 001
    OK
    127.0.0.1:6379> get key001
    "001"
    127.0.0.1:6379> getrange key001 0 1
    "00"
    127.0.0.1:6379> getset key001 002
    "001"
    127.0.0.1:6379> get key001
    "002"
    ```
- SETBIT key offset value 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)
- GETBIT key offset 对 key 所储存的字符串值，获取指定偏移量上的位(bit)
    ```java
    127.0.0.1:6379> keys *
    1) "key001"
    127.0.0.1:6379> setbit key001 3 1
    (integer) 1
    127.0.0.1:6379> getbit key001 3
    (integer) 1
    127.0.0.1:6379> getbit key001 7
    (integer) 0
    127.0.0.1:6379> getbit key002 3
    (integer) 0
    ```
- MSET key value [key value ...] 同时设置一个或多个 key-value 对
- MSETNX key value [key value ...] 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在
- MGET key1 [key2..] 获取所有(一个或多个)给定 key 的值
    ```java
    127.0.0.1:6379> keys *
    (empty list or set)
    127.0.0.1:6379> mset key001 001 key002 002
    OK
    127.0.0.1:6379> keys *
    1) "key001"
    2) "key002"
    127.0.0.1:6379> mget key001 key002
    1) "001"
    2) "002"
    127.0.0.1:6379> msetnx key002 003 key004 004
    (integer) 0
    127.0.0.1:6379> keys *
    1) "key001"
    2) "key002"
    127.0.0.1:6379> msetnx key003 003 key004 004
    (integer) 1
    127.0.0.1:6379> keys *
    1) "key004"
    2) "key003"
    3) "key001"
    4) "key002"
    ```
- SETNX key value 只有在 key 不存在时设置 key 的值
- SETEX key seconds value 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)
- SETRANGE key offset value 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始
- STRLEN key 返回 key 所储存的字符串值的长度
- PSETEX key milliseconds value 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位
    ```java
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> set key001 001
    OK
    redis-S:6379> keys *
    1) "key001"
    redis-S:6379> setnx key001 002
    (integer) 0
    redis-S:6379> get key001
    "001"
    redis-S:6379> setnx key002 002
    (integer) 1
    redis-S:6379> keys *
    1) "key001"
    2) "key002"
    redis-S:6379> setex key001 60 001A
    OK
    redis-S:6379> ttl key001
    (integer) 54
    redis-S:6379> get key001
    "001A"
    redis-S:6379> strlen key001
    (integer) 4
    redis-S:6379> psetex key002 100000 002A
    OK
    redis-S:6379> ttl key002
    (integer) 89
    redis-S:6379> get key002
    "002A"
    redis-S:6379>
    ```
- INCR key 将 key 中储存的数字值增一
- INCRBY key increment 将 key 所储存的值加上给定的增量值（increment）
- INCRBYFLOAT key increment 将 key 所储存的值加上给定的浮点增量值（increment）
- DECR key 将 key 中储存的数字值减一
- DECRBY key decrement   key 所储存的值减去给定的减量值（decrement）
    ```java
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> set key001 1
    OK
    redis-S:6379> get key001
    "1"
    redis-S:6379> incr key001
    (integer) 2
    redis-S:6379> incrby key001 3
    (integer) 5
    redis-S:6379> incrbyfloat key001 0.9
    "5.9"
    redis-S:6379> get key002
    "1"
    redis-S:6379> decr key002
    (integer) 0
    redis-S:6379> decrby key002 2
    (integer) -2
    redis-S:6379>
    ```
- APPEND key value 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾
    ```java
    redis-S:6379> keys *
    1) "key001"
    2) "key002"
    redis-S:6379> get key001
    "5.9"
    redis-S:6379> append key001 -xxx
    (integer) 7
    redis-S:6379> get key001
    "5.9-xxx"
    redis-S:6379> append key003 new-value
    (integer) 9
    redis-S:6379> keys *
    1) "key003"
    2) "key001"
    3) "key002"
    redis-S:6379> get key003
    "new-value"
    redis-S:6379>
    ```
#### Redis hash 命令

- HSET key field value 将哈希表 key 中的字段 field 的值设为 value
- HGET key field 获取存储在哈希表中指定字段的值
- HMSET key field1 value1 [field2 value2 ] 同时将多个 field-value (域-值)对设置到哈希表 key 中
- HGETALL key 获取在哈希表中指定 key 的所有字段和值
- HSETNX key field value 只有在字段 field 不存在时，设置哈希表字段的值
- HKEYS key 获取所有哈希表中的字段
- HVALS key 获取哈希表中所有值
    ```java
    redis-S:6379> hset hashTest id dog
    (integer) 1
    redis-S:6379> hget hashTest id
    "dog"
    redis-S:6379> hmset hashTest name dog_1 food bond_1
    OK
    redis-S:6379> hgetall hashTest
    1) "id"
    2) "dog"
    3) "name"
    4) "dog_1"
    5) "food"
    6) "bond_1"
    redis-S:6379> hsetnx hashTest id cat
    (integer) 0
    redis-S:6379> hgetall hashTest
    1) "id"
    2) "dog"
    3) "name"
    4) "dog_1"
    5) "food"
    6) "bond_1"
    redis-S:6379> hkeys hashTest
    1) "id"
    2) "name"
    3) "food"
    redis-S:6379> hvals hashTest
    1) "dog"
    2) "dog_1"
    3) "bond_1"
    redis-S:6379>
    ```
- HLEN key 获取哈希表中字段的数量
- HDEL key field1 [field2] 删除一个或多个哈希表字段
- HEXISTS key field 查看哈希表 key 中，指定的字段是否存在
- HMGET key field1 [field2] 获取所有给定字段的值
    ```java
    redis-S:6379> keys *
    1) "hashTest"
    redis-S:6379> hlen hashTest
    (integer) 3
    redis-S:6379> hdel hashTest id
    (integer) 1
    redis-S:6379> hlen hashTest
    (integer) 2
    redis-S:6379> hgetall hashTest
    1) "name"
    2) "dog_1"
    3) "food"
    4) "bond_1"
    redis-S:6379> hexists hashTest id
    (integer) 0
    redis-S:6379> hexists hashTest name
    (integer) 1
    redis-S:6379> hmget hashTest name food
    1) "dog_1"
    2) "bond_1"
    redis-S:6379>
    ```
- HINCRBY key field increment 为哈希表 key 中的指定字段的整数值加上增量 increment 
- HINCRBYFLOAT key field increment 为哈希表 key 中的指定字段的浮点数值加上增量 increment 
    ```java
    redis-S:6379> keys *
    1) "hashTest"
    redis-S:6379> hmset hashTest002 f1 1 f2 2 f3 3
    OK
    redis-S:6379> hgetall hashTest002
    1) "f1"
    2) "1"
    3) "f2"
    4) "2"
    5) "f3"
    6) "3"
    redis-S:6379> hincrby hashTest002 f3 2
    (integer) 5
    redis-S:6379> hincrbyfloat hashTest002 f1 1.1
    "2.1"
    ```
- HSCAN key cursor [MATCH pattern] [COUNT count] 迭代哈希表中的键值对
    ```java
    #SCAN 命令是一个基于游标的迭代器（cursor based iterator）： SCAN 命令每次被调用之后， 都会向用户返回一个新的游标， 用户在下次迭代时需要使用这个新游标作为 SCAN 命令的游标参数， 以此来延续之前的迭代过程。
    #当 SCAN 命令的游标参数被设置为 0 时， 服务器将开始一次新的迭代， 而当服务器向用户返回值为 0 的游标时， 表示迭代已结束
    # match 可以返回匹配的键值对
    # COUNT 选项的作用就是让用户告知迭代命令， 在每次迭代中应该从数据集里返回多少元素
    # 虽然 COUNT 选项只是对增量式迭代命令的一种提示（hint），但是在大多数情况下， 这种提示都是有效的（数据量少时不生效）
    redis-S:6379> hscan hashTest002 0
    1) "0"
    2) 1) "f1"
    2) "2.1"
    3) "f2"
    4) "2"
    5) "f3"
    6) "5"
    redis-S:6379> hscan hashTest002 0 match *2
    1) "0"
    2) 1) "f2"
    2) "2"
    redis-S:6379> hscan hashTest002 0 match *2 count 3
    1) "0"
    2) 1) "f2"
    2) "2"
    redis-S:6379>
    ```

#### Redis list 命令
- RPUSH key value1 [value2] 在列表中添加一个或多个值
- LPUSH key value1 [value2] 将一个或多个值插入到列表头部
- LRANGE key start stop 获取列表指定范围内的元素
- LPOP key 移除并获取列表的第一个元素
- RPOP key 移除并获取列表最后一个元素
- LPUSHX key value 将一个值插入到已存在的列表头部
- RPUSHX key value 为已存在的列表添加值
- LLEN key 获取列表长度
    ```java
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> rpush listTest 001
    (integer) 1
    redis-S:6379> lrange listTest 0 -1
    1) "001"
    redis-S:6379> lpush listTest 002
    (integer) 2
    redis-S:6379> lrange listTest 0 -1
    1) "002"
    2) "001"
    redis-S:6379> lpop listTest
    "002"
    redis-S:6379> rpop listTest
    "001"
    redis-S:6379> lrange listTest 0 -1
    (empty list or set)
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> lpushx listTest 003
    (integer) 0
    redis-S:6379> rpush listTest 001
    (integer) 1
    redis-S:6379> lpushx listTest 003
    (integer) 2
    redis-S:6379> lrange listTest 0 -1
    1) "003"
    2) "001"
    redis-S:6379> rpushx listTest 004
    (integer) 3
    redis-S:6379> lrange listTest 0 -1
    1) "003"
    2) "001"
    3) "004"
    redis-S:6379> llen listTest
    (integer) 3
    redis-S:6379>
    ```
- LINSERT key BEFORE|AFTER pivot value 在列表的元素前或者后插入元素(如果命令执行成功，返回插入操作完成之后，列表的长度。 如果没有找到指定元素 ，返回 -1 。 如果 key 不存在或为空列表，返回 0)
- LINDEX key index 通过索引获取列表中的元素
    ```java
    redis-S:6379> lrange listTest 0 -1
    1) "003"
    2) "001"
    3) "004"
    redis-S:6379> lindex listTest 1
    "001"
    redis-S:6379> linsert listTest before "001" "009"
    (integer) 4
    redis-S:6379> lrange listTest 0 -1
    1) "003"
    2) "009"
    3) "001"
    4) "004"
    redis-S:6379> linsert listTest after "001" "006"
    (integer) 5
    redis-S:6379> lrange listTest 0 -1
    1) "003"
    2) "009"
    3) "001"
    4) "006"
    5) "004"
    redis-S:6379>
    ```
- LREM key count value 移除列表元素
    - Redis Lrem 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素
    - count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 
    - count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值
    - scount = 0 : 移除表中所有与 VALUE 相等的值
    ```java
    redis-S:6379> lrange listTest 0 -1
    1) "003"
    2) "009"
    3) "001"
    4) "006"
    5) "004"
    6) "003"
    7) "003"
    8) "001"
    9) "001"
    10) "003"
    redis-S:6379> lrem listTest 2 003
    (integer) 2
    redis-S:6379> lrange listTest 0 -1
    1) "009"
    2) "001"
    3) "006"
    4) "004"
    5) "003"
    6) "001"
    7) "001"
    8) "003"
    redis-S:6379> lrem listTest 2 003
    (integer) 2
    redis-S:6379> lrange listTest 0 -1
    1) "009"
    2) "001"
    3) "006"
    4) "004"
    5) "001"
    6) "001"
    redis-S:6379> lrem listTest 0 001
    (integer) 3
    redis-S:6379> lrange listTest 0 -1
    1) "009"
    2) "006"
    3) "004"
    redis-S:6379>
    ```
- LSET key index value 通过索引设置列表元素的值
- LTRIM key start stop 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
    ```java
    redis-S:6379> lrange listTest 0 -1
    1) "009"
    2) "006"
    3) "004"
    redis-S:6379> lset listTest 1 008
    OK
    redis-S:6379> lrange listTest 0 -1
    1) "009"
    2) "008"
    3) "004"
    redis-S:6379> ltrim listTest 1 -1
    OK
    redis-S:6379> lrange listTest 0 -1
    1) "008"
    2) "004"
    redis-S:6379>
    ```
- BLPOP key1 [key2 ] timeout 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
- BRPOP key1 [key2 ] timeout 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
- RPOPLPUSH source destination 移除列表的最后一个元素，并将该元素添加到另一个列表并返回
- BRPOPLPUSH source destination timeout 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
![Redis list commond](/imgs/redis/list_commond.png)

#### Redis set 命令
- SADD key member1 [member2] 向集合添加一个或多个成员
- SMEMBERS key 返回集合中的所有成员
- SREM key member1 [member2] 移除集合中一个或多个成员
- SISMEMBER key member 判断 member 元素是否是集合 key 的成员
    ```java
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> sadd setTest 001 002 003
    (integer) 3
    redis-S:6379> smembers setTest
    1) "003"
    2) "002"
    3) "001"
    redis-S:6379> srem setTest 001
    (integer) 1
    redis-S:6379> smembers setTest
    1) "003"
    2) "002"
    redis-S:6379> sismember setTest 002
    (integer) 1
    redis-S:6379> sismember setTest 005
    (integer) 0
    redis-S:6379>
    ```
- SPOP key 移除并返回集合中的一个随机元素
- SRANDMEMBER key [count] 返回集合中一个或多个随机数
- SCARD key 获取集合的成员数
    ```java
    redis-S:6379> keys *
    1) "setTest"
    redis-S:6379> smembers setTest
    1) "003"
    2) "002"
    redis-S:6379> sadd setTest 009 007 003
    (integer) 2
    redis-S:6379> smembers setTest
    1) "009"
    2) "003"
    3) "002"
    4) "007"
    redis-S:6379> spop setTest
    "009"
    redis-S:6379> srandmember setTest 2
    1) "003"
    2) "007"
    redis-S:6379> smembers setTest
    1) "002"
    2) "003"
    3) "007"
    redis-S:6379> scard setTest
    (integer) 3
    redis-S:6379>
    ```
- SDIFF key1 [key2] 返回给定所有集合的差集
- SINTER key1 [key2] 返回给定所有集合的交集
- SUNION key1 [key2] 返回所有给定集合的并集
    ```java
    redis-S:6379> keys *
    1) "setTest002"
    2) "setTest"
    redis-S:6379> smembers setTest
    1) "yyy"
    2) "002"
    3) "003"
    4) "ttt"
    5) "007"
    redis-S:6379> smembers setTest002
    1) "xxx"
    2) "005"
    3) "004"
    4) "009"
    5) "007"
    6) "008"
    7) "006"
    8) "002"
    9) "003"
    10) "001"
    redis-S:6379> sdiff setTest setTest002
    1) "yyy"
    2) "ttt"
    redis-S:6379> sinter setTest setTest002
    1) "002"
    2) "003"
    3) "007"
    redis-S:6379> sunion setTest setTest002
    1) "xxx"
    2) "005"
    3) "004"
    4) "009"
    5) "007"
    6) "ttt"
    7) "008"
    8) "006"
    9) "yyy"
    10) "002"
    11) "001"
    12) "003"
    redis-S:6379>
    ```
- SDIFFSTORE destination key1 [key2] 返回给定所有集合的差集并存储在 destination 中
- SINTERSTORE destination key1 [key2] 返回给定所有集合的交集并存储在 destination 中
- SUNIONSTORE destination key1 [key2] 所有给定集合的并集存储在 destination 集合中
    ```java
    redis-S:6379> keys *
    1) "setTest002"
    2) "setTest"
    redis-S:6379> sdiffstore dest setTest setTest002
    (integer) 2
    redis-S:6379> smembers dest
    1) "yyy"
    2) "ttt"
    redis-S:6379> srem dest yyy ttt
    (integer) 2
    redis-S:6379> smembers dest
    (empty list or set)
    redis-S:6379> sinterstore dest setTest setTest002
    (integer) 3
    redis-S:6379> smembers dest
    1) "003"
    2) "002"
    3) "007"
    redis-S:6379> srem dest 003 002 007
    (integer) 3
    redis-S:6379> sunionstore dest setTest setTest002
    (integer) 12
    redis-S:6379> smembers dest
    1) "xxx"
    2) "005"
    3) "004"
    4) "009"
    5) "007"
    6) "ttt"
    7) "008"
    8) "006"
    9) "yyy"
    10) "002"
    11) "001"
    12) "003"
    redis-S:6379>
    ```
- SMOVE source destination member 将 member 元素从 source 集合移动到 destination 集合s
- SSCAN key cursor [MATCH pattern] [COUNT count] 迭代集合中的元素
    ```java
    redis-S:6379> keys *
    1) "setTest002"
    2) "setTest"
    redis-S:6379> smembers setTest
    1) "yyy"
    2) "002"
    3) "003"
    4) "ttt"
    5) "007"
    redis-S:6379> smembers setTest002
    1) "xxx"
    2) "005"
    3) "004"
    4) "009"
    5) "007"
    6) "008"
    7) "006"
    8) "002"
    9) "003"
    10) "001"
    redis-S:6379> smove setTest setTest002 ttt
    (integer) 1
    redis-S:6379> smembers setTest
    1) "yyy"
    2) "002"
    3) "003"
    4) "007"
    redis-S:6379> smembers setTest002
    1) "xxx"
    2) "005"
    3) "004"
    4) "009"
    5) "ttt"
    6) "007"
    7) "008"
    8) "006"
    9) "002"
    10) "003"
    11) "001"
    redis-S:6379> sscan setTest002 0 match 00* count 9
    1) "15"
    2) 1) "006"
    2) "002"
    3) "009"
    4) "003"
    5) "001"
    6) "005"
    7) "004"
    8) "007"
    9) "008"
    ```
#### Redis sorted set 命令
- ZADD key score1 member1 [score2 member2] 向有序集合添加一个或多个成员，或者更新已存在成员的分数
- ZCARD key 获取有序集合的成员数
- ZSCORE key member 返回有序集中，成员的分数值
- ZCOUNT key min max 计算在有序集合中指定区间分数的成员数
- ZINCRBY key increment member 有序集合中对指定成员的分数加上增量 increment
- ZRANK key member 返回有序集合中指定成员的索引
- ZREM key member [member ...] 移除有序集合中的一个或多个成员s
- ZRANGE key start stop [WITHSCORES] 通过索引区间返回有序集合成指定区间内的成员
    ```java
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> zadd zsetTest 1 aaa 2 bbb 4 ddd 3 ccc
    (integer) 4
    redis-S:6379> zcard zsetTest
    (integer) 4
    redis-S:6379> zscore zsetTest aaa
    "1"
    redis-S:6379> zcount zsetTest 1 3
    (integer) 3
    redis-S:6379> zincrby zsetTest 8 aaa
    "9"
    redis-S:6379> zrank zsetTest aaa
    (integer) 3
    redis-S:6379> zrem zsetTest ddd
    (integer) 1
    redis-S:6379> zrange zsetTest 0 -1
    1) "bbb"
    2) "ccc"
    3) "aaa"
    redis-S:6379>
    ```
- ZLEXCOUNT key min max 在有序集合中计算指定字典区间内成员数量
- ZRANGEBYLEX key min max [LIMIT offset count] 通过字典区间返回有序集合的成员
    - 当有序集合的所有成员都具有相同的分值时， 有序集合的元素会根据成员的字典序（lexicographical ordering）来进行排序， 而这个命令则可以返回给定的有序集合键 key 中， 值介于 min 和 max 之间的成员
    - 合法的 min 和 max 参数必须包含 ( 或者 [ ， 其中 ( 表示开区间（指定的值不会被包含在范围之内）， 而 [ 则表示闭区间（指定的值会被包含在范围之内）
    - 特殊值 + 和 - 在 min 参数以及 max 参数中具有特殊的意义， 其中 + 表示正无限， 而 - 表示负无限。 因此， 向一个所有成员的分值都相同的有序集合发送命令 ZRANGEBYLEX <zset> - + ， 命令将返回有序集合中的所有元素
    ```java
    redis-S:6379> zadd zsetTest 0 a 0 b 0 c 0 d 0 e 0 f 0 g
    (integer) 7
    redis-S:6379> zlexcount zsetTest - +
    (integer) 7
    redis-S:6379> zrange zsetTest 0 -1
    1) "a"
    2) "b"
    3) "c"
    4) "d"
    5) "e"
    6) "f"
    7) "g"
    redis-S:6379> zrangebylex zsetTest - (d
    1) "a"
    2) "b"
    3) "c"
    redis-S:6379> zrangebylex zsetTest [d +
    1) "d"
    2) "e"
    3) "f"
    4) "g"
    redis-S:6379>
    ```
- ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 通过分数返回有序集合指定区间内的成员
    ```java
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> zadd zsetTest 11 aaa 22 bbb 33 ccc 44 ddd
    (integer) 4
    redis-S:6379> zrange zsetTest 0 -1
    1) "aaa"
    2) "bbb"
    3) "ccc"
    4) "ddd"
    redis-S:6379> zrangebyscore zsetTest -inf +inf
    1) "aaa"
    2) "bbb"
    3) "ccc"
    4) "ddd"
    redis-S:6379> zrangebyscore zsetTest -inf 22
    1) "aaa"
    2) "bbb"
    redis-S:6379> zrangebyscore zsetTest  (22 +inf
    1) "ccc"
    2) "ddd"
    redis-S:6379> zrangebyscore zsetTest  (22 +inf withscores
    1) "ccc"
    2) "33"
    3) "ddd"
    4) "44"
    redis-S:6379>
    ```
- ZINTERSTORE destination numkeys key [key ...] 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
- ZUNIONSTORE destination numkeys key [key ...] 计算给定的一个或多个有序集的并集，并存储在新的 key 中
    ```java
    redis-S:6379> zrange zsetTest 0 -1
    1) "aaa"
    2) "bbb"
    3) "ccc"
    4) "ddd"
    redis-S:6379> zadd zsetTest002 11 aaa 66 yyy
    (integer) 2
    redis-S:6379> ZINTERSTORE dest 2 zsetTest zsetTest002
    (integer) 1
    redis-S:6379> zrange dest 0 -1
    1) "aaa"
    redis-S:6379> zrange dest 0 -1 withscores
    1) "aaa"
    2) "22"
    redis-S:6379> zunionstore dest2 2 zsetTest zsetTest002
    (integer) 5
    redis-S:6379> zrange dest2 0 -1 withscores
    1) "aaa"
    2) "22"
    3) "bbb"
    4) "22"
    5) "ccc"
    6) "33"
    7) "ddd"
    8) "44"
    9) "yyy"
    10) "66"
    redis-S:6379>
    ```
- ZREMRANGEBYLEX key min max 移除有序集合中给定的字典区间的所有成员
- ZREMRANGEBYRANK key start stop 移除有序集合中给定的排名区间的所有成员
- ZREMRANGEBYSCORE key min max 移除有序集合中给定的分数区间的所有成员
    ```java
    redis-S:6379> zrange zsetTest003 0 -1
    1) "a"
    2) "b"
    3) "c"
    4) "d"
    5) "e"
    6) "f"
    7) "g"
    redis-S:6379> zremrangebylex zsetTest003 (c [f
    (integer) 3
    redis-S:6379> zrange zsetTest003 0 -1
    1) "a"
    2) "b"
    3) "c"
    4) "g"
    redis-S:6379> zrange zsetTest 0 -1 withscores
    1) "aaa"
    2) "11"
    3) "bbb"
    4) "22"
    5) "ccc"
    6) "33"
    7) "ddd"
    8) "44"
    redis-S:6379> zremrangebyrank zsetTest 1 2
    (integer) 2
    redis-S:6379> zrange zsetTest 0 -1 withscores
    1) "aaa"
    2) "11"
    3) "ddd"
    4) "44"
    redis-S:6379> zremrangebyscore zsetTest 11 33
    (integer) 1
    redis-S:6379> zrange zsetTest 0 -1 withscores
    1) "ddd"
    2) "44"
    redis-S:6379>
    ```
- ZRANGE key start stop [WITHSCORES] 通过索引区间返回有序集合成指定区间内的成员
- ZREVRANGE key start stop [WITHSCORES] 返回有序集中指定区间内的成员，通过索引，分数从高到底
- ZREVRANGEBYSCORE key max min [WITHSCORES] 返回有序集中指定分数区间内的成员，分数从高到低排序
- ZREVRANK key member 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
    ```java
    redis-S:6379> zrange zsetTest 0 -1 withscores
    1) "aa"
    2) "11"
    3) "bb"
    4) "22"
    5) "cc"
    6) "33"
    7) "ddd"
    8) "44"
    redis-S:6379> zrevrange zsetTest 0 -1 withscores
    1) "ddd"
    2) "44"
    3) "cc"
    4) "33"
    5) "bb"
    6) "22"
    7) "aa"
    8) "11"
    redis-S:6379> zrevrangebyscore zsetTest 33 11
    1) "cc"
    2) "bb"
    3) "aa"
    redis-S:6379> zrevrank zsetTest ddd
    (integer) 0
    redis-S:6379> zrevrank zsetTest aa
    (integer) 3
    redis-S:6379>
    ```
- ZSCAN key cursor [MATCH pattern] [COUNT count] 迭代有序集合中的元素（包括元素成员和元素分值）
    ```java
    redis-S:6379> zrevrange zsetTest 0 -1 withscores
    1) "ddd"
    2) "44"
    3) "cc"
    4) "33"
    5) "bb"
    6) "22"
    7) "aa"
    8) "11"
    redis-S:6379> zscan zsetTest 0 match dd*
    1) "0"
    2) 1) "ddd"
    2) "44"
    redis-S:6379>
    ```
#### Redis HyperLogLog 命令
- PFADD key element [element ...] 添加指定元素到 HyperLogLog 中
- PFCOUNT key [key ...] 返回给定 HyperLogLog 的基数估算值
- PFMERGE destkey sourcekey [sourcekey ...] 将多个 HyperLogLog 合并为一个 HyperLogLog
    ```java
    redis-S:6379> keys *
    (empty list or set)
    redis-S:6379> pfadd hylog a b c d e f g h i j
    (integer) 1
    redis-S:6379> pfcount hylog
    (integer) 10
    redis-S:6379> pfadd hylog2 h i j k l m
    (integer) 1
    redis-S:6379> pfmerge hylog hylog2
    OK
    redis-S:6379> pfcount hylog
    (integer) 13
    redis-S:6379> pfcount hylog2
    (integer) 6
    redis-S:6379> pfmerge hylog3 hylog hylog2
    OK
    redis-S:6379> pfcount hylog3
    (integer) 13
    redis-S:6379>
    ```
#### reference
- [http://www.redis.cn/commands](http://www.redis.cn/commands)
- [Redis 字符串命令](http://www.runoob.com/redis/redis-strings.html)
- [Redis hash 命令](http://www.runoob.com/redis/redis-hashes.html)
- [Redis list 命令](http://www.runoob.com/redis/redis-lists.html)
- [Redis set 命令](http://www.runoob.com/redis/redis-sets.html)
- [Redis zset 命令](http://www.runoob.com/redis/redis-sorted-sets.html)
- [Redis HyperLogLog 命令](http://www.runoob.com/redis/redis-hyperloglog.html)

### Redis 持久化
- 什么是持久化 Redis的数据操作都是在内存中进行的，如果服务挂掉的话，数据会丢失。所谓持久化，就是将redis保存在内存中的数据，异步保存到磁盘上，以便在需要的时候，对数据进行恢复。
- redis数据持久化方式  快照(RDB); 写日志(AOF)
- Redis 还可以同时使用 AOF 持久化和 RDB 持久化。 在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整
- 持久化功能是可以关闭的，让数据只在服务器运行时存在
#### RDB
- 什么是RDB

    RDB Redis Database File, 将Redis内存中的数据，完整的生成一个快照，以二进制格式文件（.rdb文件）保存在硬盘当中。当需要进行恢复时，再从硬盘加载到内存中。

   ![Redis rdbs](/imgs/redis/6_redis_rdb.png)

- RDB的触发方式
    - save(同步)

        ![Redis rdbs](/imgs/redis/7_redis_rdb_save.png)
        - save命令触发RDB快照持久化，命令会阻塞Redis服务，直至rdb文件生成结束
        - 如果已经存在rdb文件，则新生成的rdb文件会替换旧的rdb文件
    - bgsave(异步)

        ![Redis rdbs](/imgs/redis/8_redis_rdb_bgsave.png)
        - bgsave命令在触发RDB持久化时，在fork()函数产生子进程的过程中，依然有可能短暂阻塞Redis服务
        - fork()出子进程，会消耗额外的内存，但是基本不会阻塞redis命令
    - config(满足条件，自动触发)
        - 在redis配置文件中，添加如下配置，在满足条件时，会自动触发RDB快照
        - 一般不推荐配置自动触发RDB快照的持久化方式
        ```python
        # 配置自动生成规则。一般不建议配置自动生成RDB文件
        save 900 1      #900秒内改变1条数据，自动生成RDB文件
        save 300 10     #300秒内改变10条数据，自动生成RDB文件
        save 60 10000   #60秒内改变1万条数据，自动生成RDB文件
        # 指定rdb文件名
        dbfilename dump.rdb
        # 指定rdb文件目录
        dir /opt/redis/data
        # bgsave发生错误，停止写入
        stop-writes-on-bgsave-error yes
        # rdb文件采用压缩格式
        rdbcompression yes
        # 对rdb文件进行校验
        rdbchecksum yes
        ```

#### AOF
- 什么是AOF

    AOF Append Only File, 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。
    ![Redis rdbs](/imgs/redis/9_redis_aof.png)
    - 所有的写入命令会追加到aof_buf（缓冲区）中
    - AOF缓冲区根据配置的策略向硬盘做同步操作
    - 随着AOF文件越来越大，需要定期对AOF文件进行重写，重写可以压缩AOF文件,也能使文件被更快的加载
    - 当Redis服务重启时，可以加载AOF文件进行数据恢复
- 命令写入
    - Redis执行写命令，将命令刷新到硬盘缓冲区当中
    - 所有写入命令都包含追加操作，直接采用协议格式，避免二次处理开销
- 缓冲同步
    - always 命令写入缓冲区后，立即调用同步操作，将写命令追加到AOF文件，追加完成后，线程返回
    - everysec 让缓冲区中的数据每秒刷新一次到AOF文件，相比always，在高写入量的情况下，可以保护硬盘。出现故障可能会丢失一秒数据(这也是AOF的默认策略)
    - no 刷新策略让操作系统来决定
- AOF重写
    - 随着时间的推移，命令的逐步写入。AOF文件也会逐渐变大。当我们用AOF来恢复时会很慢，而且当文件无限增大时，对硬盘的管理，对写入的速度也会有产生影响。Redis当然考虑到这个问题，所以就有了AOF重写
    - 原生AOF
    ```shell
    set hello world
    set hello java
    set hello python
    incr counter
    incr counter
    rpush mylist a
    rpush mylist b
    rpush mylist c
    过期数据
    ```
    - 重写后的AOF
    ```shell
    set hello python
    set counter 2
    rpush mylist a b c
    ```
    - AOF重写的方式
        - bgrewriteaof 类似于RDB快照中，bgsave的执行过程；redis客户端向Redis发bgrewriteaof命令，redis服务端fork一个子进程去完成AOF重写。这里的AOF重写，是将Redis内存中的数据进行一次回溯，回溯成AOF文件。而不是重写AOF文件生成新的AOF文件去替换

            ![Redis rdbs](/imgs/redis/10_redis_aof_rewrite.png)
        - aof重写配置
        auto-aof-rewrite-min-size：AOF文件重写需要的尺寸
        auto-aof-rewrite-percentage：AOF文件增长率
        redis提供了aof_current_size和aof_base_size，分别用来统计AOF当前尺寸（单位：字节）和AOF上次启动和重写的尺寸（单位：字节）
        AOF自动重写的触发时机，同时满足以下两点：
        - aof_current_size > auto-aof-rewrite-min-size
        - aof_current_size - aof_base_size/aof_base_size > auto-aof-rewrite-percentage
        ```python
        # 开启正常AOF的append刷盘操作
        appendonly yes
        # AOF文件名
        appendfilename "appendonly.aof"
        # 每秒刷盘
        appendfsync everysec
        # 文件目录
        dir /opt/redis/data
        # AOF重写增长率
        auto-aof-rewrite-percentage 100
        # AOF重写最小尺寸
        auto-aof-rewrite-min-size 64mb
        # AOF重写期间是否暂停append操作。AOF重写非常消耗磁盘性能，而正常的AOF过程中也会往磁盘刷数据。
        # 通常偏向考虑性能，设为yes。万一重写失败了，这期间正常AOF的数据会丢失，因为我们选择了重写期间放弃了正常AOF刷盘。
        no-appendfsync-on-rewrite yes
        ```
#### 持久化最佳策略
- RDB最佳策略
    - 建议关闭RDB，无论是Redis主节点，还是从节点，都建议关掉RDB。但是关掉不是绝对的，主从复制时还是会借助RDB
    - 用作数据备份，RDB虽然是很重的操作，但是对数据备份很有作用。文件大小比较小，可以按天或按小时进行数据备份
    - 在极个别的场景下，需要在从节点开RDB，可以再本地保存这样子的一个历史的RDB文件。虽然从节点不进行读写，但是Redis往往单机多部署，由于RDB是个很重的操作，所以还是会对CPU、硬盘和内存造成一定影响。根据实际需求进行设定
- AOF最佳策略
    - 建议开启AOF，如果Redis数据只是用作数据源的缓存，并且缓存丢失后从数据源重新加载不会对数据源造成太大压力，这种情况下，AOF可以关
    - AOF重写集中管理，单机多部署情况下，发生大量fork可能会内存爆满
    - everysec 建议采用每秒刷盘策略
#### reference
- [http://redisdoc.com/topic/persistence.html](http://redisdoc.com/topic/persistence.html)
- [Redis持久化](https://segmentfault.com/a/1190000012316003)

### Redis 实现分布式锁
#### 单机模式的Redis分布式锁
- 优缺点
    - 实现比较轻，大多数时候能满足需求；因为是单机单实例部署，如果redis服务宕机，那么所有需要获取分布式锁的地方均无法获取锁，将全部阻塞，需要做好降级处理。
    - 当锁过期后，执行任务的进程还没有执行完，但是锁因为自动过期已经解锁，可能被其它进程重新加锁，这就造成多个进程同时获取到了锁。
- 实现代码
  ```java
    import redis.clients.jedis.Jedis;
    import java.util.Collections;

    public class JedisDistributedLock {

        private static final String LOCK_SUCCESS = "OK";
        private static final Long RELEASE_SUCCESS = 1L;

        private static final String SET_IF_NOT_EXIST = "NX";
        private static final String SET_WITH_EXPIRE_TIME = "PX";

        // 获取锁，不设置超时时间
        public static boolean getLock(Jedis jedis, String lockKey, String requestId){
            String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST);
            if(LOCK_SUCCESS.equals(result)){
                return true;
            }
            return false;
        }

        // 获取锁, 设置超时时间，单位为毫秒
        public static boolean getLock(Jedis jedis, String lockKey, String requestId, Long expireTime){

            /**
            * jedis.set(key, value, nxxx, expx, time)
            *
            * Set the string value as value of the key. The string can't be longer than 1073741824 bytes (1
            * GB).
            * @param key
            * @param value
            * @param NXXX NX|XX, NX -- Only set the key if it does not already exist. XX -- Only set the key if it already exist.
            * @param EXPX EX|PX, expire time units: EX = seconds; PX = milliseconds
            *
            * @return Status code reply set成功，返回 OK
            */
            String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
            if(LOCK_SUCCESS.equals(result)){
                return true;
            }
            return false;
        }

        //释放锁
        public static boolean releaseLock(Jedis jedis, String lockKey, String requestId){

            // Lua脚本
            String script = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
            Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
            if(RELEASE_SUCCESS.equals(result)){
                return true;
            }
            return false;
        }
    }
  ```
#### 集群模式的Redis分布式锁 Redlock
- 优缺点
    - Redlock是Redis的作者antirez给出的集群模式的Redis分布式锁，它基于N个完全独立的Redis节点
    - 部分节点宕机，依然可以保证锁的可用性
    - 当某个节点宕机后，又立即重启了，可能会出现两个客户端同时持有同一把锁，如果节点设置了持久化，出现这种情况的几率会降低
    - 和单机模式Redis锁相比，实现难度要大一些
- 实现代码
    - 搭建redis集群
    ```shell
    # 安装指定版本的redis
    cd /opt
    wget http://download.redis.io/releases/redis-3.2.8.tar.gz
    tar xzf redis-3.2.8.tar.gz && rm redis-3.2.8.tar.gz
    cd redis-3.2.8
    make
    ```
    ```shell
    # 构建redis集群
    cd
    mkdir cluster-test
    cd cluster-test
    mkdir conf logs bin
    # 拷贝客户端服务器文件
    rsync -avp /opt/redis-3.2.8/src/redis-server bin/
    rsync -avp /opt/redis-3.2.8/src/redis-cli bin/
    # 配置文件
    for i in 7000 7001 7002 7003 7004 7005; do
    cat << EOF > conf/${i}.conf
    port ${i}
    cluster-enabled yes
    cluster-config-file nodes-${i}.conf
    cluster-node-timeout 5000
    appendonly yes
    logfile "logs/${i}.log"
    appendfilename "appendonly-${i}.aof"
    EOF
    done
    # 启动
    cd ${HOME}/cluster-test
    for i in 7000 7001 7002 7003 7004 7005; do
    ./bin/redis-server ./conf/${i}.conf &
    done
    # 创建集群
    sudo apt-get install -y ruby rubygems
    gem install redis
    cd /opt/redis-3.2.8/src/
    ./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
    # 测试
    cd ${HOME}/cluster-test/
    ./bin/redis-cli -c -p 7000 cluser nodes
    # 停止
    #cd ${HOME}/cluster-test
    #for i in 7000 7001 7002 7003 7004 7005; do
    #./bin/redis-cli -p ${i} shutdown && echo "redis ${i} 已停止"
    #done
    ```
    脚本执行后，出现如下日志，说明集群搭建成功
    ```shell
    >>> Creating cluster
    >>> Performing hash slots allocation on 6 nodes...
    Using 3 masters:
    127.0.0.1:7000
    127.0.0.1:7001
    127.0.0.1:7002
    Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
    Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
    Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
    M: d0e3588845a839052d9e611853740edd3b348966 127.0.0.1:7000
    slots:0-5460 (5461 slots) master
    M: 4f5518e78c1ea1c25bab0d0147f12c0281fd5f96 127.0.0.1:7001
    slots:5461-10922 (5462 slots) master
    M: 5724fb03c4527389be0d556c5dc5419a12d367c5 127.0.0.1:7002
    slots:10923-16383 (5461 slots) master
    S: 1206f7a76b8050b10e2e602fad422faee95efdb5 127.0.0.1:7003
    replicates d0e3588845a839052d9e611853740edd3b348966
    S: 8f421f529c86cefbf033e1e3a1687a9767802bd2 127.0.0.1:7004
    replicates 4f5518e78c1ea1c25bab0d0147f12c0281fd5f96
    S: d94710251235efde7da4073cc4ff104d9298bd0f 127.0.0.1:7005
    replicates 5724fb03c4527389be0d556c5dc5419a12d367c5
    Can I set the above configuration? (type 'yes' to accept): yes
    >>> Nodes configuration updated
    >>> Assign a different config epoch to each node
    >>> Sending CLUSTER MEET messages to join the cluster
    Waiting for the cluster to join...
    >>> Performing Cluster Check (using node 127.0.0.1:7000)
    M: d0e3588845a839052d9e611853740edd3b348966 127.0.0.1:7000
    slots:0-5460 (5461 slots) master
    1 additional replica(s)
    S: 8f421f529c86cefbf033e1e3a1687a9767802bd2 127.0.0.1:7004
    slots: (0 slots) slave
    replicates 4f5518e78c1ea1c25bab0d0147f12c0281fd5f96
    S: 1206f7a76b8050b10e2e602fad422faee95efdb5 127.0.0.1:7003
    slots: (0 slots) slave
    replicates d0e3588845a839052d9e611853740edd3b348966
    M: 4f5518e78c1ea1c25bab0d0147f12c0281fd5f96 127.0.0.1:7001
    slots:5461-10922 (5462 slots) master
    1 additional replica(s)
    S: d94710251235efde7da4073cc4ff104d9298bd0f 127.0.0.1:7005
    slots: (0 slots) slave
    replicates 5724fb03c4527389be0d556c5dc5419a12d367c5
    M: 5724fb03c4527389be0d556c5dc5419a12d367c5 127.0.0.1:7002
    slots:10923-16383 (5461 slots) master
    1 additional replica(s)
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.
    ```
    - 使用Redission构建redLock
    ```java
    import org.redisson.Redisson;
    import org.redisson.api.RAtomicLong;
    import org.redisson.api.RedissonClient;
    import org.redisson.config.Config;

    /**
    * Created by Haiyoung on 2018/8/11.
    */
    public class RedissonManager {

        private static final String RAtomicName = "genId_";

        private static Config config = new Config();

        private static RedissonClient redisson = null;

        public static void init(){
            try{
                config.useClusterServers()
                        .setScanInterval(200000)//设置集群状态扫描间隔
                        .setMasterConnectionPoolSize(10000)//设置对于master节点的连接池中连接数最大为10000
                        .setSlaveConnectionPoolSize(10000)//设置对于slave节点的连接池中连接数最大为500
                        .setIdleConnectionTimeout(10000)//如果当前连接池里的连接数量超过了最小空闲连接数，而同时有连接空闲时间超过了该数值，那么这些连接将会自动被关闭，并从连接池里去掉。时间单位是毫秒。
                        .setConnectTimeout(30000)//同任何节点建立连接时的等待超时。时间单位是毫秒。
                        .setTimeout(3000)//等待节点回复命令的时间。该时间从命令发送成功时开始计时。
                        .setRetryInterval(3000)//当与某个节点的连接断开时，等待与其重新建立连接的时间间隔。时间单位是毫秒。
                        .addNodeAddress("redis://127.0.0.1:7000","redis://127.0.0.1:7001","redis://127.0.0.1:7002","redis://127.0.0.1:7003","redis://127.0.0.1:7004","redis://127.0.0.1:7005");
                redisson = Redisson.create(config);

                RAtomicLong atomicLong = redisson.getAtomicLong(RAtomicName);
                atomicLong.set(1);//自增设置为从1开始
            }catch (Exception e){
                e.printStackTrace();
            }
        }

        public static RedissonClient getRedisson(){
            if(redisson == null){
                RedissonManager.init(); //初始化
            }
            return redisson;
        }

        /** 获取redis中的原子ID */
        public static Long nextID(){
            RAtomicLong atomicLong = getRedisson().getAtomicLong(RAtomicName);
            atomicLong.incrementAndGet();
            return atomicLong.get();
        }
    }
    ```

    ```java
    import org.redisson.api.RLock;
    import org.redisson.api.RedissonClient;

    import java.util.concurrent.TimeUnit;

    /**
    * Created by Haiyoung on 2018/8/11.
    */
    public class DistributedRedisLock {

        private static RedissonClient redisson = RedissonManager.getRedisson();
        private static final String LOCK_TITLE = "redisLock_";

        public static void acquire(String lockName){
            String key = LOCK_TITLE + lockName;
            RLock mylock = redisson.getLock(key);
            mylock.lock(2, TimeUnit.MINUTES); //lock提供带timeout参数，timeout结束强制解锁，防止死锁
            System.err.println("======lock======"+Thread.currentThread().getName());
        }

        public static void release(String lockName){
            String key = LOCK_TITLE + lockName;
            RLock mylock = redisson.getLock(key);
            mylock.unlock();
            System.err.println("======unlock======"+Thread.currentThread().getName());
        }

        public static void main(String[] args){
            for (int i = 0; i < 3; i++) {
                Thread t = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            String key = "test123";
                            DistributedRedisLock.acquire(key);
                            Thread.sleep(1000); //获得锁之后可以进行相应的处理
                            System.err.println("======获得锁后进行相应的操作======");
                            DistributedRedisLock.release(key);
                            System.err.println("=============================");
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                });
                t.start();
            }
        }
    }
    ```
    执行main函数，进行测试：
    ```java
    //建立的集群关系如下所示：
    12:41:14.749 [redisson-netty-1-7] DEBUG org.redisson.cluster.ClusterConnectionManager - cluster nodes state from 127.0.0.1/127.0.0.1:7001:
    d0e3588845a839052d9e611853740edd3b348966 127.0.0.1:7000 master - 0 1539319273461 1 connected 0-5460
    d94710251235efde7da4073cc4ff104d9298bd0f 127.0.0.1:7005 slave 5724fb03c4527389be0d556c5dc5419a12d367c5 0 1539319273963 6 connected
    5724fb03c4527389be0d556c5dc5419a12d367c5 127.0.0.1:7002 master - 0 1539319274465 3 connected 10923-16383
    8f421f529c86cefbf033e1e3a1687a9767802bd2 127.0.0.1:7004 slave 4f5518e78c1ea1c25bab0d0147f12c0281fd5f96 0 1539319272458 5 connected
    1206f7a76b8050b10e2e602fad422faee95efdb5 127.0.0.1:7003 slave d0e3588845a839052d9e611853740edd3b348966 0 1539319272961 4 connected
    4f5518e78c1ea1c25bab0d0147f12c0281fd5f96 127.0.0.1:7001 myself,master - 0 0 2 connected 5461-10922
    ```
    ```java
    //加锁解锁过程
    ======lock======Thread-2
    10:27:50.086 [redisson-netty-1-6] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:20] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=1, freeSubscribeConnectionsCounter=50, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@788625756 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x9109c84f, L:/127.0.0.1:49276 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:50.096 [Thread-1] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:50.102 [redisson-netty-1-6] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=32, freeConnectionsCounter=10000, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@615544583 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x088086f4, L:/127.0.0.1:49194 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:50.120 [Thread-1] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:50.120 [Thread-3] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:50.121 [Thread-1] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:20] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@1475932758 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x37cce38a, L:/127.0.0.1:49230 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:50.121 [Thread-3] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=30, freeConnectionsCounter=9998, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@1231431314 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0xea1b90d2, L:/127.0.0.1:49286 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:50.126 [redisson-netty-1-1] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:20] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@1475932758 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x37cce38a, L:/127.0.0.1:49230 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:50.127 [redisson-netty-1-1] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=32, freeConnectionsCounter=10000, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@1231431314 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0xea1b90d2, L:/127.0.0.1:49286 - R:127.0.0.1/127.0.0.1:7002]]
    ======获得锁后进行相应的操作======
    10:27:51.081 [Thread-2] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:51.082 [Thread-2] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('publish', KEYS[2], ARGV[1]); return 1; end;if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then return nil;end; local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); if (counter > 0) then redis.call('pexpire', KEYS[1], ARGV[2]); return 0; else redis.call('del', KEYS[1]); redis.call('publish', KEYS[2], ARGV[1]); return 1; end; return nil;, 2, redisLock_test123, redisson_lock__channel:{redisLock_test123}, 0, 30000, 20e96666-d967-4d3c-ac94-abeb14493160:21] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@697105324 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x3b2eb2ff, L:/127.0.0.1:49244 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:51.086 [redisson-netty-1-5] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('publish', KEYS[2], ARGV[1]); return 1; end;if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then return nil;end; local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); if (counter > 0) then redis.call('pexpire', KEYS[1], ARGV[2]); return 0; else redis.call('del', KEYS[1]); redis.call('publish', KEYS[2], ARGV[1]); return 1; end; return nil;, 2, redisLock_test123, redisson_lock__channel:{redisLock_test123}, 0, 30000, 20e96666-d967-4d3c-ac94-abeb14493160:21] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=32, freeConnectionsCounter=10000, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@697105324 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x3b2eb2ff, L:/127.0.0.1:49244 - R:127.0.0.1/127.0.0.1:7002]]
    ======unlock======Thread-2
    =============================
    10:27:51.090 [Thread-1] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:51.090 [Thread-3] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:51.091 [Thread-1] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:20] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=30, freeConnectionsCounter=9998, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@489743733 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0xc2cb57a1, L:/127.0.0.1:49204 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:51.091 [Thread-3] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=30, freeConnectionsCounter=9998, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@562253068 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x69c6d83a, L:/127.0.0.1:49260 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:51.095 [redisson-netty-1-1] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@562253068 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x69c6d83a, L:/127.0.0.1:49260 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:51.096 [redisson-netty-1-6] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:20] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=32, freeConnectionsCounter=10000, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@489743733 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0xc2cb57a1, L:/127.0.0.1:49204 - R:127.0.0.1/127.0.0.1:7002]]
    ======lock======Thread-1
    ======获得锁后进行相应的操作======
    10:27:52.102 [Thread-1] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:52.103 [Thread-1] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('publish', KEYS[2], ARGV[1]); return 1; end;if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then return nil;end; local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); if (counter > 0) then redis.call('pexpire', KEYS[1], ARGV[2]); return 0; else redis.call('del', KEYS[1]); redis.call('publish', KEYS[2], ARGV[1]); return 1; end; return nil;, 2, redisLock_test123, redisson_lock__channel:{redisLock_test123}, 0, 30000, 20e96666-d967-4d3c-ac94-abeb14493160:20] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@2085319794 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x7b0b77c0, L:/127.0.0.1:49250 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:52.107 [redisson-netty-1-5] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('publish', KEYS[2], ARGV[1]); return 1; end;if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then return nil;end; local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); if (counter > 0) then redis.call('pexpire', KEYS[1], ARGV[2]); return 0; else redis.call('del', KEYS[1]); redis.call('publish', KEYS[2], ARGV[1]); return 1; end; return nil;, 2, redisLock_test123, redisson_lock__channel:{redisLock_test123}, 0, 30000, 20e96666-d967-4d3c-ac94-abeb14493160:20] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=32, freeConnectionsCounter=10000, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@2085319794 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x7b0b77c0, L:/127.0.0.1:49250 - R:127.0.0.1/127.0.0.1:7002]]
    ======unlock======Thread-1
    =============================
    10:27:52.111 [Thread-3] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    10:27:52.112 [Thread-3] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@1360268292 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0xe28d264d, L:/127.0.0.1:49406 - R:127.0.0.1/127.0.0.1:7002]]
    10:27:52.115 [redisson-netty-1-7] DEBUG org.redisson.command.CommandAsyncService - connection released for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1); redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; return redis.call('pttl', KEYS[1]);, 1, redisLock_test123, 120000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=32, freeConnectionsCounter=10000, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using connection RedisConnection@1360268292 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0xe28d264d, L:/127.0.0.1:49406 - R:127.0.0.1/127.0.0.1:7002]]
    ======lock======Thread-3
    10:27:53.120 [Thread-3] DEBUG org.redisson.cluster.ClusterConnectionManager - slot 12809 for redisLock_test123
    ======获得锁后进行相应的操作======
    10:27:53.121 [Thread-3] DEBUG org.redisson.command.CommandAsyncService - acquired connection for command (EVAL) and params [if (redis.call('exists', KEYS[1]) == 0) then redis.call('publish', KEYS[2], ARGV[1]); return 1; end;if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then return nil;end; local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); if (counter > 0) then redis.call('pexpire', KEYS[1], ARGV[2]); return 0; else redis.call('del', KEYS[1]); redis.call('publish', KEYS[2], ARGV[1]); return 1; end; return nil;, 2, redisLock_test123, redisson_lock__channel:{redisLock_test123}, 0, 30000, 20e96666-d967-4d3c-ac94-abeb14493160:22] from slot NodeSource [slot=null, addr=null, redisClient=null, redirect=null, entry=MasterSlaveEntry [masterEntry=[freeSubscribeConnectionsAmount=0, freeSubscribeConnectionsCounter=49, freeConnectionsAmount=31, freeConnectionsCounter=9999, freezed=false, freezeReason=null, client=[addr=redis://127.0.0.1:7002], nodeType=MASTER, firstFail=0]]] using node 127.0.0.1/127.0.0.1:7002... RedisConnection@1735465853 [redisClient=[addr=redis://127.0.0.1:7002], channel=[id: 0x669d66fa, L:/127.0.0.1:49254 - R:127.0.0.1/127.0.0.1:7002]]
    ======unlock======Thread-3
    =============================
    ```
    
    
   