<!-- TOC -->

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
    - [reference](#reference)

<!-- /TOC -->

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

由于Redis对windows的支持不友好，所以这儿介绍使用docker容器来启动 redis
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
#### reference
- [Docker 安装 Redis](http://www.runoob.com/docker/docker-install-redis.html)
- [Docker 容器启动 redis](https://www.yuque.com/haiyoung/useful_notes/rpb8zg)