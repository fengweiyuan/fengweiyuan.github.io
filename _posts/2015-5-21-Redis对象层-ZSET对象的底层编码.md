---
layout:     post
title:      Zset对象的底层编码
subtitle:   Redis对象层
date:       2015-05-21
author:     冯伟源
# header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - redis
    - 数据结构
---

ZSET对象的底层编码
===

- 可以使用ziplist与skiplist作为底层编码；
- 有序集合对象的每个元素，成员都是一个字符串对象，分值都是一个double类型的浮点数；
- 这又是对象依赖对象的一个例子；

#### 1. ziplist

- 使用压缩列表作为底层实现，那么，每个集合元素使用两个相邻的ziplist节点来保存；
- 第一个节点保存元素的成员(member)，第二个元素保存元素的分值(score)；
- ziplist内的集合元素，按分值从小到大进行排序；
- 分值较小的元素靠表头，分值较大的靠表尾；

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/7D4A22478B2D4827B008193A14EFA419)

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/AF35566EDE56490AB1D34817ADABD63D)
> zset对象的各个元素，在ziplist中按分值从小到大排列

#### 2. skiplist

- 有序集合对象，不是直接使用skiplist数据结构做底层实现；
- 有序集合对象，使用了zset这个数据结构，而数据结构，是为了包含一个zskiplist与字典；

```
// zset结构体
typedef  struct  zset {
    zskiplist  *zsl;
    dict *dict;
}
```

- zsl跳跃表按从小到大，保存了有序集合的所有元素；
- 跳跃表节点的object属性保存了元素的成员，score属性则保存了元素的分值；
- 通过跳跃表，程序可以比较容易地实施范围型操作，比如ZRANK、ZRANGE等；
- dict为有序集合对象保存了其成员到分值的映射，字典的键保存了元素的成员，字典的值保存了元素的分值。通过dict，程序可以用O(1)的复杂度查找给定成员的分值，如ZSCORE命令就是用了字典；
- zset虽然同时使用了zsl与dict来保存有序集合的元素，但因为都是通过指针来共享相同的成员与分值，所以同时使用也不会浪费额外的内存，顶多多了点管理，这算是以空间换效率的一种做法了；
- 理论上，只用跳跃表，或者只用字典，都可以实现有序集合。只是说，一起用效能更好。
- 如果只用字典，对于每次范围型操作，程序都要对所有元素排序。
- 如果只用跳跃链表，查找成员分值会从O(1)上升到O(logN)时间复杂度。

#### 3. 编码转换

- 当有序集合对象可以同时满足以下两个条件，有序集合对象使用ziplist编码，否则使用skiplist编码。
  - 有序集合保存的元素数量小于128个；
  - 有序集合保存的所有元素成员的长度都小于64字节；
  ```
  redis.conf中可通过下面两个参数修改条件
  zset-max-ziplist-entries 128
  zset-max-ziplist-value 64
  ```

#### 4. zset命令在不同编码下的实现逻辑

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/482134BEF46C4E5F9BEC4E5EF01BDB1B)