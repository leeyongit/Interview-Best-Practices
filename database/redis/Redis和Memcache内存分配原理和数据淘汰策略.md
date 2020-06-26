# Redis和Memcache内存分配原理和数据淘汰策略
## 内存分配原理
Memcache采用slab table的方式分配内存，首先把可得的内存按照不同的大小来分类，在使用时根据需求找到最接近于需求大小的块分配，来减少内存碎片，但是这需要进行合理配置才能达到效果。

Redis为了屏蔽不同平台之间的差异及统计内存占用量等，对内存分配函数进行了一层封装，在程序中统一使用zmalloc、zfree系列函数，这些函数位于zmalloc.h/zmalloc.c文件中。封装就是为了屏蔽底层平台的差异，同时方便自己实现相关的统计函数。具体的实现方式如下：
- 若系统中存在Google的TC_MALLOC库，则使用tc_malloc一族的函数代替原本的malloc一族的函数。
- 若当前系统是Mac系统，则使用系统的内存分配函数。
- 对于其它情况，在每一段分配好的空间前面同时多分配一个定长的字段，用来记录分配的空间大小，通过这种方式来实现简单有效的内存分配。


## 数据淘汰策略
Redis提供了丰富的淘汰策略，包括maxmemory、maxmemory-policy、volatile-lru、allkeys-lru、volatile-random、allkeys-random、volatile-ttl、noeviction(return error)等。
- voltile-lru 从已经设置过期时间的数据集中挑选最近最少使用的数据淘汰
- voltile-ttl 从已经设置过期时间的数据库集当中挑选将要过期的数据
- voltile-random 从已经设置过期时间的数据集任意选择淘汰数据
- allkeys-lru 从数据集中挑选最近最少使用的数据淘汰
- allkeys-random 从数据集中任意选择淘汰的数据
- no-eviction 禁止驱逐数据

Memecache在容量达到指定值后，就基于LRU（Least Recently Used）算法自动删除不使用的缓存。在某些情况下LRU机制反倒会带来麻烦，会将不期待的数据从内存中清除，在这种情况下启动Memcache时，可以通过“M”参数禁止LRU算法。


