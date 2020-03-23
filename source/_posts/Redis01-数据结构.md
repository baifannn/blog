---
title: redis01-数据结构
date: 2020-03-23 21:00:49
tags:
- redis
categories:
- redis学习
---

### redis数据结构简介

| 结构类型         | 结构存储的值                                   | 结构的读写能力                                           |
| ---------------- | ---------------------------------------------- | -------------------------------------------------------- |
| STRING           | 字符串、整数、浮点数                           | 整个字符串或者一部分操作                                 |
| LIST             | 链表，链表上面每个节点包含一个字符串           | 链表的两端推入或者弹出                                   |
| SET              | 包含字符串的无需收集器，而且每个字符串唯一     | 添加获取移出单个元素，检查单个是否存在，计算并集交集差集 |
| HASH             | 键值对的无序散列表                             | 添加获取移出单个键值对；获取所有键值对                   |
| ZSET（有序集合） | 字符串与浮点数分值之间的有序集合，排序根据分值 | 添加获取删除单个元素，根据分值范围获取元素               |

#### 字符串STRING

类似字符串

| 命令 | 行为                   |
| ---- | ---------------------- |
| GET  | 获取存储在给定健中的值 |
| SET  | 设置存储在给定健中的值 |
| DEL  | 删除存储在给定健中的值 |

```bash
$ redis-cli
redis 127.0.0.1:6379> set hello world
OK
redis 127.0.0.1:6379> get hello
"world"
redis 127.0.0.1:6379> del hello
(integer)1
redis 127.0.0.1:6379> get hello 
(nil)
```

#### 列表LIST

链表结构

| 命令   | 行为                                     |
| ------ | ---------------------------------------- |
| RPUSH  | 将给定值推入列表的右端                   |
| LRANGE | 设置存储在给定健中的值                   |
| LINDEX | 删除存储在给定健中的值                   |
| LPOP   | 从列表的左端弹出一个值，并返回被弹出的值 |

```bash
$ redis-cli
redis 127.0.0.1:6379> rpush list-key item
(integer)1
redis 127.0.0.1:6379> rpush list-key item2
(integer)2
redis 127.0.0.1:6379> rpush list-key item
(integer)3
redis 127.0.0.1:6379> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"
redis 127.0.0.1:6379> lindex list-key 1
"item2"
redis 127.0.0.1:6379> lpop list-key
"item"
redis 127.0.0.1:6379> lrange list-key 0 -1
1) "item"
2) "item2"
```



#### 集合SET

集合是无序，不可能像使用列表，将元素推入集合或者弹出集合

| 命令      | 行为                                     |
| --------- | ---------------------------------------- |
| SADD      | 将给定值推入列表的右端                   |
| SMMEMBERS | 返回集合包含的所有元素                   |
| SISMEMBER | 检查给定元素是否存在于集合中             |
| SREM      | 从列表的左端弹出一个值，并返回被弹出的值 |

```bash
$ redis-cli
redis 127.0.0.1:6379> sadd set-key item
(integer)1
redis 127.0.0.1:6379> sadd list-key item2
(integer)1
redis 127.0.0.1:6379> sadd list-key item3
(integer)1
redis 127.0.0.1:6379> sadd list-key item
(integer)0
redis 127.0.0.1:6379> smembers set-key
1) "item"
2) "item2"
3) "item3"
redis 127.0.0.1:6379> sismember set-key item4
(integer)0
redis 127.0.0.1:6379> sismember set-key item
(integer)1
redis 127.0.0.1:6379> srem set-key item2
(integer)1
redis 127.0.0.1:6379> srem set-key item2
(integer)0
redis 127.0.0.1:6379> smembers set-key
1) "item"
2) "item3"
```

#### 散列HASH

可以存储多个键值对。存储的值可以为字符串或者数字。散列在很多方面像一个微缩版本的redis，不少字符串命令都有相应的散列版本

| 命令    | 行为                                     |
| ------- | ---------------------------------------- |
| HSET    | 在散列里面关联给定的键值对               |
| HGET    | 获取指定散列键的值                       |
| HGETALL | 检查给定元素是否存在于集合中             |
| HDEL    | 如果给定键存在于散列里面，那么移除这个健 |

```bash
$ redis-cli
redis 127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 1
redis 127.0.0.1:6379> hset hash-key sub-key2 value2
(integer) 1
redis 127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 0
redis 127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"
redis 127.0.0.1:6379> hdel hash-key sub-key2
(integer) 1
redis 127.0.0.1:6379> hdel hash-key sub-key2
(integer) 0
redis 127.0.0.1:6379> hget hash-key sub-key1
"value1"
redis 127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
```

#### 有序集合ZSET

有序集合的键称之为member，值称之为分值score。有序集合是Redis里面唯一一个即可以根据成员访问元素，又可以根据分值以及分值的排列顺序来访问元素的接口。

| 命令    | 行为                                                       |
| ------- | ---------------------------------------------------------- |
| ZADD    | 将一个带有给定分值的成员添加到有序集合里面                 |
| ZRANGE  | 根据元素在有序排列中所出的位置，从有序集合里面获取多个元素 |
| HGETALL | 获取有序集合在给定分值范围内的所有元素                     |
| ZREM    | 如果给定成员存在于有序集合，那么移除这个成员               |

```bash
$ redis-cli
redis 127.0.0.1:6379> zadd zset-key 728 member1
(integer) 1
redis 127.0.0.1:6379> zadd zset-key 982 member0
(integer) 1
redis 127.0.0.1:6379> zadd zset-key 982 member0
(integer) 0
redis 127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member2"
4) "982"
redis 127.0.0.1:6379> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"
redis 127.0.0.1:6379> zrem zset-key member1
(integer) 1
redis 127.0.0.1:6379> zrem zset-key member1
(integer) 0
redis 127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member2"
2) "982"
```

#### 

