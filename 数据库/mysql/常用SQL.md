# 常用 SQL
## 查询当前执行时间超过 60 s 的事务
```SQL
select *
from information_schema.innodb_trx
where TIME_TO_SEC(timediff(now(), trx_started)) > 60;
```
## 查看事务隔离级别
```SQL
show variables like '%tx_isolation%';
```
## 三元运算符
```SQL
select a from  t where a = if(2 > 1, 2, 1);
```
## on duplicate key update(key is (a, b))
```SQL
insert into t (a, b, c)
values ('a1', 'b1', 'c1'),
       ('a2', 'b2', 'c2')
on duplicate key update 
        c  = IF(values(c) != '', values(c), '');
```
## key 相关
```SQL
alter table student add key key_name_age(name, age);
alter table student drop key key_name_age;
show keys from t;
```