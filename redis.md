<!-- TOC -->

- [Redis 基础](#redis-%E5%9F%BA%E7%A1%80)
    - [Redis 是什么？](#redis-%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F)
    - [Redis 数据结构](#redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
        - [value 对应的五种数据结构](#value-%E5%AF%B9%E5%BA%94%E7%9A%84%E4%BA%94%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
        - [Redis 核心对象 redisObject](#redis-%E6%A0%B8%E5%BF%83%E5%AF%B9%E8%B1%A1-redisobject)
        - [Redis 五种数据结构对应的内部编码](#redis-%E4%BA%94%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AF%B9%E5%BA%94%E7%9A%84%E5%86%85%E9%83%A8%E7%BC%96%E7%A0%81)
    - [reference](#reference)

<!-- /TOC -->

### Redis 基础

#### Redis 是什么？
Redis是一个开源（BSD许可）的，利用内存进行存储的数据结构存储系统；它可以用作数据库、缓存和消息中间件。
- redis由意大利人 Salvatore Sanfilippo 使用C语言开发
- redis支持字符串（string）、列表（list）、集合（set）、有序集合（zset）、散列表（hash）五种基本数据结构类型
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
- redisObject 对象的数据类型，对应五种数据类型
- redisObject 对象的编码方式，指定所绑定数据类型的编码方式
- 




##### Redis 五种数据结构对应的内部编码

![Redis 五种数据结构对应的内部编码](/imgs/redis/5_data_encoding.PNG)




#### reference
- [redis中文官网](http://www.redis.cn/)
- http://www.runoob.com/redis/redis-tutorial.html