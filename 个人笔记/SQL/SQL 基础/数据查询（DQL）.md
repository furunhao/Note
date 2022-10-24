##### 使用表达式查询

```sql
-- （注意，这里的查询操作并不会改变数据库里的值） || ： 连接符号，像 Java 中连接字符串的 + 号
select productid,
       productname,
       productprice || ' * ' || 1.25 || ' = ' || productprice * 1.25 as newprice
from productinfo;
```

##### 过滤重复数据
```sql
select distinct productid as 过滤重复id,
                productid,
                productname,
                productprice
from productinfo;
```

##### 查询指定字段的数据

```sql
elect first_name,
       last_name
from contacts;

-- 可以指定某个模式下的表或视图的列;   模式.表名.字段；这样可以查询其他模式下的表
select code.contacts.contact_id,
       contacts.first_name,
       contacts.last_name
from code.contacts;
```

##### order by

```sql
select
	column 1,
	column 2,
from
	table_name
order by
	column 1[ASC | DESC][NULLS FIRST | NULLS LAST],
	column 1[ASC | DESC][NULLS FIRST | NULLS LAST],
	……;
```

```sql
-- 默认以 ASC 升序展示，加上 DeSC 降序显示
select product_id,
       product_name,
       standard_cost,
       list_price
from products
order by list_price desc;
-- 优先把空值展示在顶部
select *
from productinfo
order by origin nulls first;
-- 优先把空值展示在底部
select *
from productinfo
order by origin nulls last;
-- 使用查询列表中字段的位置（数字代表字段在表里的位置）来作为排序字段，这么做一是为了方便，二是为了防止使用UNION时出现错误。
select *
from productinfo
order by 7 desc;
-- 多字段排序：当第一个字段数据相同时，才会对第二个字段排序
select *
from productinfo
order by 1 desc,
         2 desc,
         3 desc,
         4 desc,
         5 desc,
         6 desc,
         origin nulls last;
```

##### where 筛选

```sql
select product_id,
       product_name,
       list_price
from products
where list_price > 3000
order by list_price desc;
-- <>  要求 list_price 字段下不等于 8867.99 的数据 ; 等同于 != (不等于）
select product_id,
       product_name,
       list_price
from products
where list_price > 3000
  and list_price != 5499.99;

select product_id,
       product_name,
       list_price
from products;
-- 使用 substr 函数列出前几位等于 intel 的数据
select product_id,
       product_name,
       list_price
from products
where substr(product_name, 0, 3) = 'AMD';
-- where 多个条件限制
select product_id,
       product_name,
       list_price
from products
where list_price <= 3500
  and list_price >= 3000;
-- 针对上面语句的简单写法(between 左边的值要比右边的小，这是一个闭区间)
select product_id,
       product_name,
       list_price
from products
where list_price between 3000 and 3200;
```

##### and

- and 连接的两个条件都为 true 才能返回数据
- and 的优先级 大于 or

```sql
select product_id,
       product_name,
       list_price
from products
where list_price < 5499.99
   and products.list_price > 8867.99;
```

##### or

```sql
select product_id,
       product_name,
       list_price
from products
where list_price < 5499.99
   or products.list_price > 8867.99;
```

##### union

- 使用 or 会使索引会失效，在数据量较大的时候查找效率较低

- 当涉及到查询多个列时，通常使用 union 代替 or

- 上面的代码可以改写为

```sql
select product_id,
       product_name,
       list_price
from products
where list_price < 5499.99
   union
select product_id,
       product_name,
       list_price
from products
where products.list_price > 8867.99;
```

##### 模糊查询

```sql
select customer_id,
       name,
       address
from customers
where name like '_Amer%'; -- 下划线代表 Amer 前面只有一个字符，百分号则不会限制
```

##### in / not in

```sql
select customer_id,
       name,
       address,
       website
from customers
where customer_id in (170, 200);
-- not in 不查询等于 2和4的数据
select customer_id,
       name,
       address,
       website
from customers
where customer_id not in (2, 4);
```

##### is

```sql
select *
from productinfo
where origin is null;
-- is 查询origin值不为空的数据
select *
from productinfo
where origin is not null;
```

##### group by

```sql
-- 分组函数 group by 允许出现在where语句后，即被分组的语句都是经过where语句筛选过的
-- 当查询中存在GROUP BY子句时，SELECT列表中只能存在分组函数，或出现在GROUP BY子句中的字段。
select product_name,
       avg(list_price)
from products
group by product_name;
-- 多个字段分组
select category          产品类型编码,
       avg(productprice) 平均价格,
       origin            产地
from productinfo
group by category,
         origin;

select *
from productinfo;
```

##### update

```sql
- 更新数据
update productinfo
set productprice = 6000
where origin = 44;
```

##### where / group by / having

```sql
-- where 和 group by 混合使用
select category          产品类型编码,
       avg(productprice) 平均价格,
       origin            产地
from productinfo
where productprice > 6000
group by category,
         origin;

-- HAVING和GROUP BY一起使用，限制搜索条件。它和WHERE子句不一样，HAVING子句与组有关，而不与单个的值有关。在GROUP BY子句中，作用于GROUP BY创建的组。
-- 过滤分组后的数据
-- where 服务于 from
-- having 服务于 group by
select category,
       avg(productprice)
from productinfo
group by category
having avg(productprice) > 6000;
```

##### 子查询

```sql
-- 子查询（嵌套查询）出现在 select update delete中；本质是where后的一个表达式
-- 单一条件查询（查询产品类型为 MP3 的数据）
select productname,
       productprice
from productinfo
where category = (select categoryid
                  from categoryinfo
                  where categoryname = 'MP3');

-- 多条件查询（查询大于最小值和小于最大值的数据）
select productname,
       productprice
from productinfo
where productprice > (select min(productprice)
                      from productinfo)
  and productprice < (select max(productprice)
                      from productinfo)

-- 子查询返回多行(in)
-- 查询 categoryid 等于 电视和MP3的数据
select productname,
       productprice
from productinfo
where category in
      (select categoryid
       from categoryinfo
       where categoryname = '电视'
          or categoryname = 'MP3');

-- ANY：表示满足子查询结果的任何一个。和＜、＜=搭配，表示小于等于列表中的最大值；而和＞、＞=配合时表示大于等于列表中的最小值。
-- 从产品表PRODUCTINFO中查询出价格低于指定价格列表中的最大值。指定的价格列表就是指产品类型编码为“0100030002”的所有产品价格
select productname,
       productprice
from productinfo
where productprice < any
      (select productprice
       from productinfo
       where category = '0100030002')
  and category <> '0100030002';

-- some 等同于 any
-- SOME的用法和ANY一样，只不过ANY多用在非“=”的环境中。SOME这里表示找出和子查询中任何价格相等的产品。
select productname,
       productprice
from productinfo
where productprice =
    some (select productprice
          from productinfo
          where category = '0100030002')
  and category <> '0100030002';

-- ALL：表示满足子查询结果的所有结果。和＜、＜=搭配，表示小于等于列表中的最小值；而和＞、＞=配合时表示大于等于列表中的最大值。
-- 找出比指定价格列表还低的产品数据
select productname,
       productprice
from productinfo
where productprice <
          all (select productprice
               from productinfo
               where category = '0100030002');
```



