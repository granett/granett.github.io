---
layout: post
title: elasticsearch5-x集群HA原理(shards、replicas)
date: 2016-06-08 11:15:06 
tag: 工具1
---

最近在搭建es5.2的高可用集群，在这个过程中加深了对es的原理理解，基本分为四个阶段
es单机—>es集群（多台机器）—>es分片和副本集分布原理—>es高可用集群
#### 1.es单机
在第一个阶段基本概念的掌握还是比较熟练的，对应着关系型数据库（mysql）来理解es：
###### 文档（document）
文档（document）是ElasticSearch中的存储形式。对所有使用ElasticSearch的案例来说，他们最终都可以归结为对文档的搜索，一个文档相当于mysql里的一条数据
###### 索引（index）
ElasticSearch将它的数据存储在一个或多个索引（index）中。索引就像数据库，可以向索引写入文档或者从索引中读取文档
###### 类型（type）
每个文档都有与之对应的类型（type）定义。这允许用户在一个索引中存储多种文档类型，比如在“资料”索引下，有pdf类型和word类型，并为不同文档提供类型提供不同的映射
###### 映射（mapping）
所有文档写进索引之前都会先进行分析，如何将输入的文本分割为词条、哪些词条又会被过滤，这种行为叫做映射（mapping）。一般由用户自己定义规则，可以理解为pdf类型的文档的映射就是pdf含有的字段

#### 2.es集群
es集群是通过多台服务器来搭建，它们拥有一个共同的clustername比如叫做“escluster”，每台服务器叫做一个节点，拥有自己的节点名字：nodename，配置文件如下：
```
集群名称，用于定义哪些elasticsearch节点属同一个集群。
cluster.name: bigdata
节点名称，用于唯一标识节点，不可重名
node.name: server3
设置索引的分片数,默认为5 
index.number_of_shards: 5 
设置索引的副本数,默认为1: 
index.number_of_replicas: 1 
```
这时多台服务器都可以对外提供查询和更改接口，他们彼此之间负载均衡，我们代码访问时可以配置为多台：
```
配置访问集群的client
TransportClient client = new PreBuiltTransportClient(settings)
      .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.254.133"), 9300));
	  .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.254.134"), 9300));
      .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.254.135"), 9300));
      .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.254.136"), 9300));
执行插入操作
client.prepareIndex("info", "info",UUID.randomUUID().toString())
		        .setSource(builder.string())
		        .get();        
```
>es集群肯定效率各方面都要比单机强很多，但是如果集群中一台机器挂掉了，我们其余的几台会不会安然无恙？而数据会不会丢失？我们目前并不能保证，所以我们要配置一套高可用的集群

#### 3.es分片和副本集分布原理
配置一套高可用的集群，我们必须要了解es集群的数据分布和负载原理
##### shards（分片）
在配置文件里我们看到默认的shards是5个，**一个索引的全部数据会被分开存储在这几个分片上，**我们用3个分片来看下效果：

**单机分片分布：**
![QQ图片20180613180955.png](https://upload-images.jianshu.io/upload_images/6073827-3ba00761992496dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**2台集群分片分布：**
![QQ图片20180613181116.png](https://upload-images.jianshu.io/upload_images/6073827-27cacc2a5d8b9efa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3台集群分片分布：**
![QQ图片20180613180539.png](https://upload-images.jianshu.io/upload_images/6073827-0b066b5d926b1f52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>**单机：**该机器节点拥有全部3个分片
**2台集群：**一个节点有一个分片，另一台有两个分片
**3台集群：**每个节点拥有一个分片

根据以上案例可以看到**es的工作原理是把整个数据分割成3个分片，然后每个节点平均分配**
当我们有9个分片时，估计是每台机器各自3个分片
![QQ图片20180613181444.png](https://upload-images.jianshu.io/upload_images/6073827-e49864aeceb3e624.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
总结
>**1.es集群只是根据我们指定的分片数来平均分配到各个节点上
2.当节点数大于分片数时，也不会冗余，只是让多余的节点空着不存数据，不分摊压力
3.当有节点挂掉时，我们一定会丢失一部分数据**

根据上面的总结，**es集群无法避免机器挂掉后仍然能不丢数据的正常运行，解决办法是配置文件的另一个参数index.number_of_replicas来达到高可用目的**

#### es高可用集群
index.number_of_replicas是索引的副本数，也就是索引的分片副本数，**我们通过3台机器3个分片的配置来看下效果**
1个副本
![QQ图片20180613182349.png](https://upload-images.jianshu.io/upload_images/6073827-16b181faacb64725.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2个副本
![QQ图片20180613182521.png](https://upload-images.jianshu.io/upload_images/6073827-c1bbd3305fa89654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3个副本
![QQ图片20180613182644.png](https://upload-images.jianshu.io/upload_images/6073827-6af0fab84eb78155.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>1.每个副本都会把当前的分片全部复制一份并平均分布到集群节点上
2.当副本数3时，由于此时每台机器都已经占满自己的3个分片了，所以此时需要增加新的机器来存放第三个副本，所以提示了Unassigned？

#### 高可用原理
我们以3机器3分片2副本为例：
![QQ图片20180613182349.png](https://upload-images.jianshu.io/upload_images/6073827-b0f691a3366be417.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**第一步：**
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在每个分片编码（0,1,2）上的边框有粗有细，粗的是主分片，细的是副本分片，当node1的机器挂掉时，主节点1丢失，此时集群由green（健康）转为red，因为主节点丢失导致。

**第二步：**
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其它节点上存在着主分片1的完整副本，所以集群立即将这些分片在 Node 2 和 Node 3 上对应的副本分片提升为主分片，此时集群的状态将会为 yellow 。为什么我们集群状态是 yellow 而不是 green 呢？虽然我们拥有所有的三个主分片，但是同时设置了每个主分片需要对应2份副本分片，而此时只存在一份副本分片。所以集群不能为 green 的状态，不过我们不必过于担心：如果我们同样关闭了 Node 2 ，我们的程序 依然 可以保持在不丢任何数据的情况下运行，因为 Node 3 为每一个分片都保留着一份副本。

**第三步：**
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果我们重新启动 Node1，Node1依然拥有着之前的分片，它将尝试去重用它们，同时仅从主分片复制发生了修改的数据文件，集群状态由yellow转为green。

**第四步：**
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果不重启Node1，而是新增了一台机器并启动加进集群，此时集群可以将缺失的副本分片再次进行分配写入到新增的Node上，那么集群的状态也将由yellow转为green。

由此我们了解了高可用的基本原理，想要配置好高可用，节点数，分片数，副本数这三个数量之间是有很紧密的联系的。

