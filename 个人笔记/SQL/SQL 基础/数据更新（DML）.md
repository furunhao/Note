> 添加用 insert; 修改用 update; 删除用 delete; 搜索用 select

>请注意，当您在应用程序中嵌入查询时，切勿使用星号 (*)。即使要从表的所有列中检索数据，显式指定要从中查询数据的列也是一种很好的做法。您应该只使用星号 (*) 速记来表示临时查询。

> 这是因为将来由于业务变化，一张表可能会有更多或更少的列。如果您在应用程序代码中使用星号 (*) 并假定表具有一组固定的列，则应用程序可能不会处理附加列或访问已删除的列。

### 搜索数据

```sql
select
	column 1,
	column 2,
from
	table_name;
	
```

### 插入数据

```sql
-- 插入数据
insert into manageinfo
(manageid,
 loginname,
 password,
 name,
 tel)
values ('1',
        'xiaowang',
        '123456',
        'liming',
        '12345678');
```

```sql
-- 通过其他表向表里插入数据(确保两张表列的个数和字段数据类型一样才可以使用）
insert into logininfo
(loginname,
 loginpassword)
select loginname,
       password
from manageinfo;
-- 目标表里的数据都会插入到新表里
```

```sql
-- 直接创建带数据的表
select *
from login;
create table login as
select loginname,
       password
from manageinfo;
insert into login
(loginname,
 password)
values ('wang',
        '123');
insert into login
(loginname,
 password)
values ('zhang',
        '123');
```

### 修改数据

```sql
update login
set loginname = 'test'; -- 不加where条件会修改loginname字段下的所有数据
update login
set loginname = 'zhang'
where password = '123456';
-- 修改条件修改指定的数据
```

### 删除数据

```sql
delete
from login; -- 删除表内所有的数据

delete
from login
where loginname = 'wang';
--删除指定的数据
-- truncate 删除数据；和delete一样，删除表里的数据，但是速度明显提高
truncate table login;
```