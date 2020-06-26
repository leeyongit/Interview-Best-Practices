# Redis 介绍

## 一 、简介
1. Redis的特点
Redis是一种非关系数据库(不会像MySQL一样多个表直接存在直接关联关系),也是一种内存型数据库
Redis是单线程的,并且是直接基于内存的,所以执行效率会非常高

2. Redis的数据结构
- String : 最普通和常用的一种结构,key/value
- Hash : 类似于HashMap,底层也是哈希表
- List : 类似于LinkedList,底层是双向链表
- Set : 类似于HashSet,于HashSet的特性一样,不允许重复数据,不记录元素添加顺序
- SortSet : 会自动的进行排序的set
- Streams(流) : 18年刚刚发布的一个新的数据结构,不是太了解,相关资料比较少

3. 各个数据类型最大存储量
原文地址：https://redis.io/topics/data-types
- Strings类型：一个String类型的value最大可以存储512M
- Lists类型：list的元素个数最多为2^32-1个，也就是4294967295个。
- Sets类型：元素个数最多为2^32-1个，也就是4294967295个。
- Hashes类型：键值对个数最多为2^32-1个，也就是4294967295个。
- Sorted sets类型：跟Sets类型相似。



