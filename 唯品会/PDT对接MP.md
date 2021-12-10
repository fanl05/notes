## 规则
分页查 com.vip.smc.query.store.service.StoreApiService.queryStores 接口，查 storeStatus = 2 的店铺 store_ids

com.vip.smc.query.store.service.StoreApiService.getStoreFestivalNotice 接口

根据传入的 storeIds 查询 queryStartDate 到 queryEndDate 时间范围内的加时，此时间长度做成配置，queryEndDate 为当前时间

StoreFestivalService 字段中，只取 changedService = "deliveryAndCollection"

调用 createOrUpdate 接口和 delete 接口，同步数据，先插入后删除，删除条件是 business_mode 是 mp 且修改时间在此次插入时间之前

最终计算结果是 endTime + lazyTime

`表字段与 MP 接口字段对应关系`
```
ext_id -> ?
business_mode -> 4
node_code -> 5260
object_code -> store_id
sec_object_code -> 
subject_code -> 
action_type -> 2
start_time -> beginTime
end_time -> endTime
inc_duration -> serviceRule.lazyHour
reason -> ?
is_deleted -> 0
source -> mp
created_by_user -> ryland
```