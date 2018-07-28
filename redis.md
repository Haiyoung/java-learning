Redis
<!-- TOC -->

- [Redis 官网](#redis-%E5%AE%98%E7%BD%91)
- [Redis 概况](#redis-%E6%A6%82%E5%86%B5)
    - [Redis 是什么？](#redis-%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F)
    - [Redis 数据结构](#redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
        - [value 对应的五种数据结构](#value-%E5%AF%B9%E5%BA%94%E7%9A%84%E4%BA%94%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
        - [Redis 核心对象 redisObject](#redis-%E6%A0%B8%E5%BF%83%E5%AF%B9%E8%B1%A1-redisobject)
            - [编码方式（encoding）](#%E7%BC%96%E7%A0%81%E6%96%B9%E5%BC%8F%EF%BC%88encoding%EF%BC%89)
        - [Redis 五种数据结构对应的内部编码](#redis-%E4%BA%94%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AF%B9%E5%BA%94%E7%9A%84%E5%86%85%E9%83%A8%E7%BC%96%E7%A0%81)
    - [reference](#reference)
- [搭建 Redis 环境](#%E6%90%AD%E5%BB%BA-redis-%E7%8E%AF%E5%A2%83)
    - [启动 redis server](#%E5%90%AF%E5%8A%A8-redis-server)
    - [启动 redis-cli](#%E5%90%AF%E5%8A%A8-redis-cli)
    - [redis-cli连接远程服务](#redis-cli%E8%BF%9E%E6%8E%A5%E8%BF%9C%E7%A8%8B%E6%9C%8D%E5%8A%A1)
    - [reference](#reference)
- [Redis命令](#redis%E5%91%BD%E4%BB%A4)
    - [Redis keys 命令](#redis-keys-%E5%91%BD%E4%BB%A4)
    - [Redis 字符串命令](#redis-%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%91%BD%E4%BB%A4)
    - [Redis hash 命令](#redis-hash-%E5%91%BD%E4%BB%A4)
    - [Redis list 命令](#redis-list-%E5%91%BD%E4%BB%A4)
    - [Redis set 命令](#redis-set-%E5%91%BD%E4%BB%A4)
    - [Redis sorted set 命令](#redis-sorted-set-%E5%91%BD%E4%BB%A4)
    - [Redis HyperLogLog 命令](#redis-hyperloglog-%E5%91%BD%E4%BB%A4)
    - [reference](#reference)

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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
    ```python
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
- ZCOUNT key min max 计算在有序集合中指定区间分数的成员数
- ZINCRBY key increment member 有序集合中对指定成员的分数加上增量 increment
- ZINTERSTORE destination numkeys key [key ...] 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
- ZLEXCOUNT key min max 在有序集合中计算指定字典区间内成员数量
- ZRANGE key start stop [WITHSCORES] 通过索引区间返回有序集合成指定区间内的成员
- ZRANGEBYLEX key min max [LIMIT offset count] 通过字典区间返回有序集合的成员
- ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 通过分数返回有序集合指定区间内的成员
- ZRANK key member 返回有序集合中指定成员的索引
- ZREM key member [member ...] 移除有序集合中的一个或多个成员
- ZREMRANGEBYLEX key min max 移除有序集合中给定的字典区间的所有成员
- ZREMRANGEBYRANK key start stop 移除有序集合中给定的排名区间的所有成员
- ZREMRANGEBYSCORE key min max 移除有序集合中给定的分数区间的所有成员
- ZREVRANGE key start stop [WITHSCORES] 返回有序集中指定区间内的成员，通过索引，分数从高到底
- ZREVRANGEBYSCORE key max min [WITHSCORES] 返回有序集中指定分数区间内的成员，分数从高到低排序
- ZREVRANK key member 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
- ZSCORE key member 返回有序集中，成员的分数值
- ZUNIONSTORE destination numkeys key [key ...] 计算给定的一个或多个有序集的并集，并存储在新的 key 中
- ZSCAN key cursor [MATCH pattern] [COUNT count] 迭代有序集合中的元素（包括元素成员和元素分值）

#### Redis HyperLogLog 命令
- PFADD key element [element ...] 添加指定元素到 HyperLogLog 中
- PFCOUNT key [key ...] 返回给定 HyperLogLog 的基数估算值
- PFMERGE destkey sourcekey [sourcekey ...] 将多个 HyperLogLog 合并为一个 HyperLogLog

#### reference
- [http://www.redis.cn/commands](http://www.redis.cn/commands)
- [Redis 字符串命令](http://www.runoob.com/redis/redis-strings.html)
- [Redis hash 命令](http://www.runoob.com/redis/redis-hashes.html)
- [Redis list 命令](http://www.runoob.com/redis/redis-lists.html)
- [Redis set 命令](http://www.runoob.com/redis/redis-sets.html)
- [Redis zset 命令](http://www.runoob.com/redis/redis-sorted-sets.html)
- [Redis HyperLogLog 命令](http://www.runoob.com/redis/redis-hyperloglog.html)