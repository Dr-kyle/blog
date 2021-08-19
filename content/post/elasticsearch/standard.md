# 规范

1. Index 命名规范

​              <team>_<business>-yyyy_MM_dd, 优点： 便于统计，分析，模板不易冲突

2. 数据写入加入 @timestamp 时间字段,格式： 时间戳 或 2021-08-06T06:46:34.258Z 

3. 基于时间序列的索引使用 [data stream](https://confluence.newegg.org/display/XABD/Data+Streams) 管理

4. 副本数默认 1

5. 关闭动态 mapping, 未指定的字段统一设为 keyword，可搜索，但是不可聚合

6. 设置解析字段的最大深度为5， eg： "a.b.c.d.e.g"， 深度为 6 的数据将无法写入, 避免嵌套过深

7. nested 字段中字段的数量限制为  10

8. nested 字段中 json object 数量为 100，避免查询数据过多时拖垮集群

9. index 最大字段数调整为 100

10. 单个shard 的大小控制在 10GB – 50 GB 之间

11. 不要把ES 当做数据库来使用，例如：更新频繁且量比较大，数据之间又有关联