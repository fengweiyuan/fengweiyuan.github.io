---
layout:     post
title:      Set对象的底层编码
subtitle:   Redis对象层
date:       2015-05-18
author:     冯伟源
# header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - redis
    - 数据结构
---

SET对象的底层编码
===

- SET对象的底层编码可以是intset或者dict；

#### 1. 使用intset做底层实现

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/C88302AEFFDA4F85BB5C910B1AAF287D?ynotemdtimestamp=1539402928162)

#### 2. 使用dict做底层实现

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/F0F940A8958D45C5804B0F46F401FAF3?ynotemdtimestamp=1539402928162)

#### 3. 编码转换

- 当集合对象可以同时满足下面两个条件时，使用intset编码，否则使用hashtable编码。
  - 集合对象保存的所有元素都是整数；
  - 集合对象保存的元素数量不超过512个；
  - redis.conf可以改条件
  ```
  # 集合对象使用ziplist编码，保存的元素超过多少个，就会转为hashtable编码
  set-max-intset-entries 512
  ```
  
```
sadd number 1 3 5
object encoding number    #intset
sadd number seven
object encoding number   #hashtable
```

#### 4. SET对象在不同编码下对命令的处理逻辑

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/C557B6A6FE9B46A1AA42E2C221526FE0?ynotemdtimestamp=1539402928162)

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/94067596D9BD4571A1ED4ED98E4853B8?ynotemdtimestamp=1539402928162)