---
layout:     post
title:      Hash对象的底层编码
subtitle:   Redis对象层
date:       2015-05-17
author:     冯伟源
# header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - redis
    - 数据结构
---

Hash对象的底层编码
===

- 当哈希对象可以同时满足以下两个条件时，会使用ziplist作为底层实现，其他情况，使用hashtable编码；
  - a. 哈希对象保存的所有键值对的key与value的字符串长度都小于64字节；
  - b. 哈希对象保存的键值对数量小于512个；
  - 这两个条件可以在redis.conf中修改。
  ```
  hash-max-ziplist-value
  hash-max-ziplist-entries
  ```


#### 1. 编码转换

- 条件本来满足，后来不满足了，对象的编码转换能自动执行；
- 会读出ziplist的所有key-value值，并转移到HT里面；
- 如果以后hash对象缩小了满足上述两个条件，也不会自动转换回ziplist，只扩不缩的原则；

```
hset people name '0123456789012345678901234567890123456789012345678901234567890123'
object encoding name #是ziplist
hset people name '01234567890123456789012345678901234567890123456789012345678901234'
object encoding name # 是hashtable
```

#### 2. 使用ziplist作为底层实现

- 会将保存了key值的ziplist结点，放到ziplist表尾。
- 然后将保存了value值的ziplist结点，再放到ziplist表尾；
- 因此，key与value值是相邻的，key在前，value在后；
- 后添加的key-value会在ziplist的表尾；

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/A2C3E1F19469409AA72097902A8A956A)
![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/C9442AC923D64C99BDE5796094930407)

#### 3. 使用字典作为底层实现

- 会使用字典作为底层实现。哈希对象的每个键值对，都用一个字典键值对来保存；
- 字典的每个键都是一个字符串对象，对象中保存了键值对的键。
- 字典的每个值都是一个字符串对象，对象中保存了键值对的值。
- 哈希对象 -> 字典数据结构 -> 字符串对象   
- 所以不一定就是对象使用编码，可能是编码也再使用对象，形成一些嵌套引用；

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/7627AE5FA9B641089907D4B16CD5BF89)