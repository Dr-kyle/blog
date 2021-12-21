# ES-Cloud Analysis

Data Size

时间段内 取最大值



Top index size 10,      getIndexStatistics

先获取最后一次收集数据的最大时间，然后根据最后一次收集的数据时间， 前缀求和，取前 10



Monitor-Index

getIndexStatistics 最大取 200

默认显示前 10



1. getIndexStatistics    index_prefix 为 -1 的问题
2. index 指标 total 折线图的问题， index rate 也有问题



index rate 

1. 使用 msearch 分开查询，相当于查询固定的 index 增长趋势



1. 按时间段分组
2. 求每个具体的 index 在这一段时间内的增长量
3. 按时间段将所有的 具体 index 总量相加就是该 别名的 增长量 







