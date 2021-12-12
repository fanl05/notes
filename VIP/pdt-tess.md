# 算法工程化
## 工程模型


## OSP 接口
```java
String forecastSupplierWarehouse(Request request);
```

## 数据结构
原始的数据数据量大且离散，进行 one-hot 编码时耗时且占用内存大，需要对数据进行压缩

将特征与转换后的数据映射关系数据加载进内存，以 map 的形式存储，每一个特征对应一个 map，以 city_code 为例，读取的数据如下：
```excel
NULL,103102106,105103128,102103108,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
0.0,0.0,0.0,0.0,...
```
转换后的 map 如下，key 为 city_code，value 为转换后的值的列表
```java
Map<String, List<Double>> provinceCodeMap
```
为提高数据的可复用性，提高内存利用率，今后增加接口预测别其他且使用该特征时，也可以以此为标准进行 one-hot 编码

## 加载策略
需要评估数据量大小
1. 若数据量不大则项目启动时从 Hive 出仓的 MySQL 表全量读取数据直接加载进内存
2. 若数据量大则参照 PDT 二级缓存，项目启动时从 Hive 出仓的 MySQL 表全量读取数据加载进 Redis，查询时优先查内存，若没有则查 Redis，并将数据同步至内存

## 更新策略
1. Saturn 任务每日定时从 Hive 出仓的表读取数据

2. Hive 数据更新后通过 VMS 传输，消费消息，更新 Redis 并广播删除内存对应的 key（无法保证数据一致性）
3. Hive 出仓到 MySQL，

## 疑问
1. CSV 数据如何变动，是每天在一个时间段统一变动还是持续变化？每次变动是大规模的变化还是个别的映射关系发生改变？

    A: 统一变动；对于某些特征是大规模变化，对于某些特征是几乎不会变
2. 如果模型更新了，但对应的映射关系没有更新，对结果有什么影响？

    A: 结果完全不对
3. TF-Serving 更新模型需要重启吗？需不需要部署多台?

    A: 需要；需要