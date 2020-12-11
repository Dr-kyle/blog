# postES Manager

```
第一目标代替cerebro
ES-portal 功能点   
1. 角色划分  某些集群管理员 超级集群管理员
1. 集群管理员查看集群状态，集群指标(集群名，node数，indices数，shards数，文档数，存储量)
2. 集群状态预警(发微信，发邮件)
3. 查看index分配情况， 未分配的shard, 已分配的shard 参考cerebro
4. 模板模块
5. 线程池的监控

Index module
Index 增长情况，写入量增长情况(建议所有写入的数据增加写入的时间 insetTime)

node module
统计每个节点上index占用的内存
每个节点上占用磁盘最大的index




```

## Index

- **request**

  create index or import data from other source

- **cerebro**

  replace cerebro 

## Cluster Manage

 add, delete, update

## Organization

- Team

  `admin`,`normal`,``

- User