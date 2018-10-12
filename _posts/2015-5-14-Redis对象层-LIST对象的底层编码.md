---
layout:     post
title:      LIST对象的底层编码
subtitle:   Redis对象层
date:       2015-05-14
author:     冯伟源
# header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - redis 数据结构
---

LIST对象的底层编码
===

#### 1. 知识

- 列表对象的编码可以是ziplist（压缩链表）或者linkedlist（双端链表）；
- 源码: `object.c`
- 使用ziplist编码的条件：
  - 列表对象保存的所有字符串元素的长度都少于64字节；
  - 列表对象保存的元素数量小于512个；
- 使用linkedlist编码的条件：
  - 不能满足ziplist编码条件的就会用linkedlist编码，对象的编码转换会自动执行；
  - 可以在redis.conf中修改自动转换临界条件：
  ```
  list-max-ziplist-entries 512
  list-max-ziplist-value 64
  ```

#### 2. 编码转换

```
rpush lst 1 2 3
object encoding lst   #ziplist
rpush lst '11111122222333334444455555666667777788888999990000011111222223333'
object encoding lst #就是linkedlist了
```

#### 3. 创建使用ziplist作为底层编码的list对象

```
robj *createZiplistObject(void) {
    unsigned char *zl = ziplistNew();          //ziplist.c创建压缩链表底层数据结构
    robj *o = createObject(REDIS_LIST,zl);     //以压缩链表数据结构作为参数，创建链表对象
    o->encoding = REDIS_ENCODING_ZIPLIST;      //指定编码
    return o;
}
```

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/00F27406155E45B796485715369B20B2?ynotemdtimestamp=1539351978490)

#### 4. 创建使用linkedlist作为底层编码的list对象

```
robj *createListObject(void) {
    list *l = listCreate();                      //adlist.c中创建双端链表底层数据结构             
    robj *o = createObject(REDIS_LIST,l);        //传入双端链表底层数据结构，创建列表对象
    listSetFreeMethod(l,decrRefCountVoid);       //为双端链表底层数据结构，设置释放的方法
    o->encoding = REDIS_ENCODING_LINKEDLIST;     //指定编码
    return o;
}
```

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/26A51D888E934CA5881E426A66BA1556?ynotemdtimestamp=1539351978490)

#### 5. 不同编码下List命令的实现

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/56BF4187ACA64138BC20740B295F9A5C?ynotemdtimestamp=1539351978490)
 

