---
layout: post
title: "Redis之String详解"
date: 2019-09-22
tag: Redis
---   

### 简介
Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。
 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 
Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。
### redis实例
![01.png][1]

redis是单线程、单进程、单实例的。一台服务器可以开启多个redis实例，通过端口区分。可以通过
```
redis-cli -p 端口号
```
来连接不通的实例，如果不指定端口号，默认是6379
redis通过epoll来读取client请求，因为是单线程、单进程，所以保证每个连接内命令的顺序一致，也就是说客户端处理好线程安全问题，交给redis之后一定是安全的。每个redis默认会生成16个库0-16，可以通过配置文件修改。可以通过
```
redis-cli -n （0-16）
```
来选择不同的库进入，或者在进入之后通过
```
select (0-16)
```
来切换库
### 数据类型
redis通过key-value的形式存放数据，value有String、List、hash、Set、sorted_set多种类型。其中key上面会保留一些属性，例如：
1. type

    value的类型，可以通过
    ```
    type key
    ```
    来获取value的数据类型

2. Object encoding

    value的编码，比如embstr（字符串），int（数字）,可以通过
    ```
    Object encoding key
    ```
    命令来获取。该方式可以加快处理速度，比如我的编码是embstr，当我执行累加操作的时候，发现不是int，就直接抛出异常， 
   不用再去判断value是否是数字了。每次执行成功执行string或int的操作之后，都会把编码保存在key中。

### String类型
![02.png][2]

String类型实际存储的是byte也就是字节，二进制安全，不涉及到编码问题。因为是byte所以redis的String实际上包含，字符串，数值，bitmap三种类型。
- 字符串

针对字符串的操作有
写入数据
```
set key value
```
获得数据
```
get key
```
追加数据
```
append key value
```
替换字符串中的字符
```
setrange key offset value
```
获取字符串中的字符，字符串为双向索引，从前到后为0-n，从后到前为-1-(-n)
```
getrange key start end
```
获取字符串长度
```
strlen key
```
- 数值

针对数值的操作有
自增1
```
incr key
```
自减1
```
decr key
```
增加n
```
incrby key n
```
减少n
```
decrby key n
```
- bitmap

bitmap，针对位进行操作，在redis中，每个字节都有自己的下标，一个字节有8位二进制，同样的每一位也有自己的下标，位的下标从0开始，一个字符串中，第一个字节是0-7，第二个字节就是8-15，可以通过
```
setbit key offset value
```
来修改key所对应的value的offset位的下标的值,如执行
```
setbit k1 1 1
setbit k1 7 1
setbit k1 9 1
setbit k1 14 1
get k1
```
此时k1的所对应的二级制值为：0100 0001 0100 0010,输出的value就是"AB"    
统计某几个字节二进制位为1的数量
```
bitcount key start end
```
其中start，end为字节的序号    
寻找某个二进制位，第一个出现的位置
```
bitpos key 1 [start] [end]
```
寻找二进制1出现的位置，start，end为字节下标
位的操作，与或非异或等
```
bitop and andkey k1 k2 ...
```
k1,k2等做与操作，结果保存为andkey,同理还有或操作
```
bitop or orkey k1 k2 ...
```
bitmap应用场景

1. 有用户系统，统计用户登录天数，且窗口随机

    365天每天占一位，每个用户是一个key，当某个用户登录后，该key对应的位数设置为1
    ```
    setbit sean 1 1
    setbit sean 7 1
    setbit sean 364 1
    STRLEN sean
    BITCOUNT sean -2 -1
    ```

    每个字节是8位，所以该命令返回结果为最后16天中，该用户登录了几天。

2. 京东618做活动：送礼物大库备货多少礼物(统计活跃用户)

    每个用户占一位，有多少用户占多少位，每个日期是一条记录，如果用户登录，将该天所对应的记录的对应位设置为1，之后将n 
   个时间做或操作，只要登录过一次就是1，然后统计出结果中二进制1的数量，就是该时间段内的活跃用户数。
    ```
    setbit 20190101   1  1
    setbit 20190102   1  1
    setbit 20190102   7  1
    bitop  or   destkey 20190101  20190102
    BITCOUNT  destkey  0 -1 
    ```

  [1]: https://www.princeleiblog.tk/usr/uploads/2019/11/3853555343.png
  [2]: https://www.princeleiblog.tk/usr/uploads/2019/11/1924639956.png









