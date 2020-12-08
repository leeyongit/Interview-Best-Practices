# MySQL Explain 执行计划

### EXPLAIN 输出格式
```ini
mysql> explain select * from user_info where id = 2\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user_info
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 8
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```
> 各列的含义如下:
> id: SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符.
> select_type: SELECT 查询的类型.
> table: 查询的是哪个表
> partitions: 匹配的分区
> type: join 类型
> possible_keys: 此次查询中可能选用的索引
> key: 此次查询中确切使用到的索引.
> ref: 哪个字段或常数与 key 一起被使用
> rows: 显示此查询一共扫描了多少行. 这个是一个估计值.
> filtered: 表示此查询条件所过滤的数据的百分比
> extra: 额外的信息