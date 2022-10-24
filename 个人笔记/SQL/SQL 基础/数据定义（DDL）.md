> 创建表

``` sql
create table productinfo
(
    productid    varchar2(10),   --商品编号
    productname  varchar2(20),   --商品名称
    productprice number(8, 2),   --商品价格
    quantity     number(10),     --商品数量
    category     varchar2(10),   --商品类型
    desperation  varchar2(1000), --商品描述
    origin       varchar2(10)--产地
);
```

> 删除表

```sql
drop table productinfo;
```

> 修改表

```sql
alter table productinfo
    add test varchar2(20); -- 添加列到表里

alter table productinfo
    modify test number; --修改字段类型

alter table productinfo
    drop column test; -- 删除字段

alter table productinfo
    add test number
    modify origin number; -- 修改多个字段

alter table productinfo
    drop (test, origin);-- 删除多个字段

alter table productinfo
    add (test number,test1 number); --添加多个字段

alter table productinfo
    modify (test varchar2(10),test1 varchar2(1)); -- 修改多个字段


```



