---
layout:     post
title:      string对象的底层编码
subtitle:   Redis对象层
date:       2015-05-12
author:     冯伟源
# header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - redis
    - 数据结构
---

string对象的底层编码
===

#### 1. REDIS_ENCODING_INT

- 又叫`长整型编码`
- 如果对象保存的是整数值，一般可以用long int类型来表示；
- 现在很多服务器上，long int是64位的，所以当你保存一个 `(-2^64/2,2^64/2)`时，会用该底层编码保存；
- 它直接依赖了C的数据类型，而没有依赖redis的任何底层支撑(如sds,intset等)；

![](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/AAF9BBC6BDA145D19F17CFE1263FCD83)


#### 2. REDIS_ENCODING_EMBSTR

- 直接使用了SDS数据结构来作为对象的底层编码；
- 字符串长度如小于等于32字节，一次分配内存，object与底层sds在内存上是连续的；

![](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/F64530A5DD6A423EBF466AD3E086D431)

#### 3. REDIS_ENCODING_RAW

- 直接使用了SDS数据结构来作为对象的底层编码；
- 所不同的是，两次分配内存，内存不相邻；
- 字符串长度大于32字节的，会使用；
- 不一定是32字节，在新版本上，宏的默认值会有所不同；

![](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/B81065D75A6845DCBB132368CC5072A7)

#### 4. 编码的转换

- int、embstr编码的字符串对象，在条件满足的情况下，会被转换为raw编码的字符串对象；
- 对key执行命令使key保存的不再是整数值，而是字符串值，就会发生int->raw；
- embstr编码的字符串对象是只读的，所以对embstr编码的字符串对象执行任何修改命令时，就会变成raw编码；

```
set number 1024
object encoding number  #原来是int编码
append number " is a good number"
object encoding number  #转换成了raw编码
```

```
set msg "hello"                   
object encoding msg   #embstr
append msg " again!"
object encoding msg  #raw
```

#### 5. 不同编码下String命令的实现

![image](https://note.youdao.com/yws/public/resource/974b6569a100fd7aa6edd53407460255/1EAECCA779B3420986C4D694B1798844)
 

