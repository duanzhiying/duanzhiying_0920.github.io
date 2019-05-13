---
title: Storm Trident WordCount入门实例
date: 2017-03-30 18:38:58
tags: [Storm,Trident]
categories: 实时计算
toc: false
---

Trident是基于Storm的高级抽象。每次发送的数据，Tuple被分成一组组的batch，每个batch分配一个唯一的事务id。不同batch之间有序，按照input stream到来的顺序。

<!--more-->

Trident提供了成熟的API来处理batches，并不是需要单个处理每个batch，而是提供方法来做聚合并且存储聚合的结果，可以存储在内存中，如Memcached、Cassandra或者其他结构。
Trident具有完全的容错机制，确保tuple植被处理一次exactly-once。
Strom Trident的实例如下。

## Trident WordCount实例

```java 
/**
 * Created by duanzhiying on 2017/3/22.
 */
public class TridentTopologyMain {
    public static void main(String[] args) throws InterruptedException {

        FixedBatchSpout spout = new FixedBatchSpout(new Fields("sentence"),3,
                new Values("my name is duanzhiying"),
                new Values("my favourite animal is dog"),
                new Values("my dream is to be a data scientist"));
        //不停循环发送
        spout.setCycle(true);

        TridentTopology topology = new TridentTopology();

        //newStream读取数据源，数据源也可以是Kafka等
        //txId表示Trident保留元数据在Zookeeper中存储的node名。对每个stream的状态都保存。
        //split是Trident已经实现了的方法，分割字符串，按照空格。此处方法可以自己定义重写
        //persistentAggregate对流进行聚合。并将结果存入TridentState类型。TridentState展现所有聚合后的结果数据
        TridentState wordCounts = topology.newStream("spout1",spout)
                .each(new Fields("sentence"),new Split(),new Fields("word"))
                .groupBy(new Fields("word"))
                .persistentAggregate(new MemoryMapState.Factory(),new Count(),new Fields("count"))
                .parallelismHint(2);

        //DRPC：distributed RPC。通过DRPC服务器协调接收一个RPC请求，发送到topology，从topology接收结果。
        //并行查询wordCounts
        //args是固定的，表示从client接收到的参数 The tuple contains one field called "args" that contains the argument provided by the client.
        LocalDRPC drpc = new LocalDRPC();
        //方法名命名为words
        topology.newDRPCStream("words", drpc)
                .each(new Fields("args"), new Split(), new Fields("word"))
                .groupBy(new Fields("word"))
                .stateQuery(wordCounts, new Fields("word"), new MapGet(), new Fields("count"))
                //如下注释掉的语句是对所有的count做累加
                /*.each(new Fields("count"), new FilterNull())
                //第一个fields是输入的字段。最后一个是聚合后的，如下sum是聚合后字段的名称
                //同样支持group by。感觉语法实现有点类似于ElasticSearch
                .aggregate(new Fields("count"), new Sum(), new Fields("sum"))*/;

        //Configuration集群配置
        Config conf = new Config();
        //打印debug信息
        conf.setDebug(false);
        //Topology run
        conf.put(Config.TOPOLOGY_MAX_SPOUT_PENDING, 1);
        //本地模式
        LocalCluster cluster = new LocalCluster();
        //提交拓扑
        cluster.submitTopology(
                "word-count",
                conf,
                topology.build() //创建拓扑
        );
        for (int i = 0; i < 100; i++) {
            System.out.println("DRPC RESULT: " + drpc.execute("words", "duanzhiying animal my you"));
            Thread.sleep(1000);
        }
    }
}
```

## 实例运行结果：
![Trident](/images/posts/2017.3.30/Trident.png)
