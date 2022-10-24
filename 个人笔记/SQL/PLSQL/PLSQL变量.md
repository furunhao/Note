## 声明变量语法结构

```sql
variable_name datatype -- variable_name：变量名称  datatype：变量数据类型
[ -- 二到五行可选
[NOT NULL] -- 非空约束
{:=|DEFAULT}expression -- 当使用 NOT NULL时，花括号里的内容为必需，表示二选一；’=‘表示赋值；DEFAULT表示默認值
];                     -- expression：表示变量存储的值，可以是表达式
```

## 声明常量语法结构

```sql
constant_name CONSTANT datatype -- 声明常量时，必须加上关键字 CONSTANT
[NOT NULL]
{:=|DEFAULT}expression;
```

- expression 后的表达式分为
  - 字符型表达式
  - 日期型表达式
  - 数值型表达式
  - 布尔值表达式
  - 直接的一个值
- 变量和常量的数据类型分为
  - 🌟标量类型变量：单一的类型，不存在组合						
  - 🌟复合类型变量：由几种单一类型组合而成的一个结构体
  - 引用类型变量：使用一个其他数据项的引用

### 标量类型的变量

- 数值类型

  - <kbd>NUMBER</kbd>表示整数和浮点数；以十进制存储；
    - 格式：NUMBER(precision,scale)；precision 表示位数，最大38位；scale 表示小数点；
    - NUMBER(3,1)：表示 -99.9~99.9 的数字范围
  - <kbd>PLS_INTEGER</kbd>表示的范围是-2147 483 648～2 147 483 647之间
  - <kbd>SIMPLE_INTEGER</kbd>是<kbd>PLS_INTEGER</kbd>的子类型；取值范围一样，不允许为空；
    - 如果数据本身不需要溢出检查而且也不可能是空，那么可以选择该类型，该类型的性能比<kbd>PLS_INTEGER</kbd>高

- 字符类型

  - <kbd>CHAR</kbd>固定长度字符串；最长32767个字节，默认为1；如果值的长度达不到定义的长度，将用空格补齐
  - <kbd>VARCHAR2（VARCHAR）</kbd>作为变量时最长32767个字节，但作为字段存储时是4000个字节
  - <kbd>NCHAR</kbd><kbd>NVARCHAR2</kbd>类型使用方式同<kbd>CHAR</kbd>和<kbd>VARCHAR2</kbd>相同，只不过它们与国家的字符集有关
  - <kbd>LONG</kbd>以可变的方式存储数据，**PL/SQL**中作为变量可表示最长可达32 760字节的字符串，如果作为存储字段则可达2GB

- 布尔类型，它不能用做定义表中的数据类型。但**PL/SQL**中该类型可以用来存储逻辑上的值。它有3个值可选：TRUE、FALSE、NULL。

- 日期类型，主要有<kbd>DATE</kbd>和<kbd>TIMESTAMP</kbd>

  - <kbd>DATE</kbd>类型可以存储月、年、日、世纪、时、分和秒。
  - <kbd>TIMESTAMP</kbd>类型由DATE演变而来，可以存储月、年、日、世纪、时、分和秒以及小数的秒。

- ☀ <kbd>%type</kbd>引用已经使用过的数据类型去给新变量定义

  - 优点：利用%TYPE定义的变量或常量数据类型都一致，当有变动的需求时，只要改变被引用的变量或常量的数据类型，其他引用处的数据类型自然就变了，避免了逐条修改的麻烦。
  - 使用PL/SQL语句块通常都是操作数据库表的数据，操作过程中避免不了出现数据的传递，这时变量利用%TYPE就可以完全兼容提取的数据，而不至于出现数据溢出或不符的情况。
  - 当利用%TYPE定义数据类型时，可以保证变量的数据类型和表中的字段类型同步，当表字段类型发生变化时，PL/SQL块变量的数据类型不需要修改。

  ```sql
  DECLARE
      v_productid            productinfo.productid%TYPE;--产品ID -- 引用productid的数据类型给 v_productid
      v_productname          VARCHAR2(20);--产品名称
      v_productprice         NUMBER(8, 2);--产品价格
      v_quantity             NUMBER(10);--数量
      v_desperation CONSTANT v_productname%TYPE := '测试';--测试 -- 定义常量，引用v_desperation的数据类型给新常量
      v_spitgr               SIMPLE_INTEGER     := 99.9;
      v_long                 LONG               := 'LONG类型测试';
      v_date                 DATE               := SYSDATE;
  BEGIN
      SELECT
          productid,
          productname,
          productprice,
          quantity
      INTO v_productid,v_productname,v_productprice,v_quantity
      FROM productinfo
      WHERE productid = '0240040001'; -- 查到的数据存到变量里
      DBMS_OUTPUT.PUT_LINE('v_productid=' || v_productid);
      DBMS_OUTPUT.PUT_LINE('v_productname=' || v_productname
          || '长度=' || LENGTH(v_productname));
      DBMS_OUTPUT.PUT_LINE('v_productprice=' || v_productprice);
      DBMS_OUTPUT.PUT_LINE('v_quantity=' || v_quantity);
      DBMS_OUTPUT.PUT_LINE('v_desperation=' || v_desperation);
      DBMS_OUTPUT.PUT_LINE('v_spitgr=' || v_spitgr);
      DBMS_OUTPUT.PUT_LINE('v_long=' || v_long);
      DBMS_OUTPUT.PUT_LINE('v_date=' || v_date);
  END;
  ```

---

### 复合类型的变量

> 一个变量包含多个元素，可以存储多个值；需要先定义，才能去声明

- PL/SQL 记录类型
  - 包含一个或多个成员，每个成员的类型也不相同；可以是标量类型，也可以是引用其他变量的类型(%type)；
  
  - 较适合处理查询语句中有多个列的情况。最常用的就是在调用某张表中的一行记录时，利用该类型变量存储这行记录。如果想要调
    用其中的数据，可以用“变量名称.成员名称”的格式进行调用。
  
  - “记录类型”有两种声明的方式
  
    - 第一种声明语法
  
    ```sql
    type type_name is record -- type_name 定义的类型名称
        (
        filed_name datatype -- 类型下的成员名称和数据类型
        [
        [NOT NULL] -- 可选，非空约束
        {:=|DEFAULT}expression -- 为记录成员赋值，expression 为赋值表达式
        ]
        [,field_name datatype[[NOT NULL]{:=|DEFAULT}expression]] -- 与前面定义成员语法相同，表现为可以有多个成员的需求
        );
    ```
  
    - demo 01
  
    ```sql
    declare
        type product_rec is record -- 声明类型变量 product_rec
                            -- 声明类型下的三个成员
                            (
                                v_product_id    products.product_id%type, -- 产品ID -- 引用已有的变量
                                v_product_name  varchar2(255),            -- 产品名称 -- 定义变量
                                v_product_price number(8, 2)              -- 价格 -- 定义变量
                            ); -- 声明行记录类型
        v_product product_rec; -- 声明变量 v_product，类型是 product_rec
    begin
        select PRODUCT_ID, PRODUCT_NAME, LIST_PRICE into v_product from PRODUCTS where PRODUCT_ID = '228';
        --这里INTO后面直接是v_product记录类型，这样赋值会依据声明记录类型时里面成员的顺序依次赋值。这种赋值方式比较方便。
        DBMS_OUTPUT.PUT_LINE('product_id=' || v_product.v_product_id); -- 输出
        DBMS_OUTPUT.PUT_LINE('product_name=' || v_product.v_product_name); -- 输出
        DBMS_OUTPUT.PUT_LINE('product_price=' || v_product.v_product_price); -- 输出
    end;
    ```
  
    - 第二种声明：利用%ROWTYPE声明记录类型数据
  
    ```sql
    declare
        v_product products%rowtype; -- 声明变量 v_product，数据类型是表 products 的行记录类型
    begin
        select * into v_product from PRODUCTS where PRODUCT_ID = '228'; -- 查询了一条记录存到 v_product
        DBMS_OUTPUT.PUT_LINE('product_id=' || v_product.product_id); -- 输出
        DBMS_OUTPUT.PUT_LINE('product_name=' || v_product.product_name); -- 输出
        DBMS_OUTPUT.PUT_LINE('product_price=' || v_product.list_price); -- 输出
    end;
    ```
  
- PL/SQL 索引表类型（关联数组）

  - 和数组类似，利用键值查找对应的值，这里的键值和数组的下标不同，索引表中下标允许使用字符串
  - 数字的长度不是固定值，可以根据需要自动增长。键值是整数或字符串。而其中的值就是普通的标量类型，也可以是记录类型。
  - 可以利用“变量名称（键值）”为其赋值或取值，如果某个键值的指向已经有数据了，那么该操作就是更改已有的数据
  
  ```sql
  -- 索引表类型本身的定义语法，并没有包含变量的定义
  declare
  type ty_name is table of -- 定义类型名称
      {
      column_type| -- 索引表中的数据类型（标量数据类型）
      variable_name%type | -- 引用类型
      table_name.column_name%type | -- 引用类型
      table_name%rowtype -- 引用表的类型
      }
      [not null]
      index by {pls_integer | binary_integer | varchar2 (v_sieze)};
  -- 如果想把某个变量声明成索引表类型，就按照这个语法
  variable_name type_name; -- variable_name 变量名称 type_name 索引表的名称
  ```
  
  - 使用数字作为键值
  
  ```sql
  -- 使用数字作为键值
  declare
      type product_a is table of products%rowtype -- 声明第一个索引表，引用 products 表的行记录
      index by binary_integer; -- 索引类型，可以认为是键值
      type product_b is table of varchar2(8) -- 声明第二个索引表，数据类型是 varchar2
      index by pls_integer;-- 索引类型，可以认为是键值
      v_prt_row product_a; -- 定义变量，类型是 product_a
      v_prt product_b; -- 定义变量，类型是 product_b
      begin
      v_prt(1):='正数'; --为变量赋值；赋值的方式同数组类似，以“变量名（索引）”的形式赋值，当取值时也需要同样的规则。
      v_prt(-1):='负数';
      select * into v_prt_row(1) from PRODUCTS where PRODUCT_ID = '228'; -- 给变量数值
      DBMS_OUTPUT.PUT_LINE('行数据-v_prt_row(1)='||v_prt_row(1).PRODUCT_ID||'---'||v_prt_row(1).PRODUCT_NAME); -- 输出
      DBMS_OUTPUT.PUT_LINE('v_prt(1)='||v_prt(1)); -- 输出
      DBMS_OUTPUT.PUT_LINE('v_prt(-1)='||v_prt(-1)); -- 输出
  end;
  ```
  
  - 使用字符串作为键值
  
  ```sql
  -- 使用字符串作为键值
  declare
      type pro_a is table of number(8) -- 声明索引表，数据类型 number
      index by varchar2(20); -- 索引类型
      v_pro_char pro_a; -- 定义变量，类型是 pro_a
      begin
      v_pro_char('test'):=123; -- 给变量赋值
      v_pro_char('test1'):=0;  -- 给变量赋值
      DBMS_OUTPUT.PUT_LINE('v_pro_char(123)='||v_pro_char('test')); -- 输出变量的值
      DBMS_OUTPUT.PUT_LINE('v_pro_char(000)='||v_pro_char('test1'));-- 输出变量的值
      DBMS_OUTPUT.PUT_LINE('v_pro_char(000)='||v_pro_char.FIRST);
      DBMS_OUTPUT.PUT_LINE('v_pro_char(000)='||v_pro_char(v_pro_char.FIRST));
  end;
  
  ```
  
  - `varray` 变长数组
    
    - 这个类型的元素个数是需要限制的，它是一个存储有序元素的集合，集合下标从1开始，适合数据较少的时候使用
    
    ```sql
    type ty_name is{varray|varying array}(size_limit) -- type_name 数组名称；
    -- varray 和 varying array 必须二选一，表示数组类型；size_limit：数组长度
    of element_type[not null] -- element_type 数组里元素的类型
    ```
    
    - demo 01
    
    ```sql
    declare
        type varr is varray(100) of varchar2(20); -- 定义数组 varr，长度100；varchar2 类型，长度20
        v_product varr:=varr('1','2'); -- 声明变量，类型是varr,初始化了两个元素
        begin
        v_product(1):='this is a'; -- 给第一个数组元素赋值
        v_product(2):='this is b'; -- 给第二个数组元素赋值
        DBMS_OUTPUT.PUT_LINE('v_product(1)='||v_product(1)); -- 输出
        DBMS_OUTPUT.PUT_LINE('v_product(2)='||v_product(2)); -- 输出
    end;
    ```

## 表达式

### 数值表达式

```sql
declare
    v_result number(10,4); -- 定义变量的数据类型，长度十位，保留4位小数
    begin
    v_result:=sqrt(58+25*3+(19-9)**2); -- 给变量赋值
    DBMS_OUTPUT.PUT_LINE('平方根='||v_result); -- 输出；其中的“||”用于连接两个字符串，这里隐式地把数值型转换成了字符串类型
end;
```

### 关系表达式

## 结构控制

#### if 结构

```sql
IF condition THEN -- then 不能省
statements; -- 当 condition 为 true 时，才会执行 statements
END IF;
```

```sql
declare
    v_result number(10,4); -- 定义变量的数据类型，长度十位，保留4位小数
    begin
    v_result:=sqrt(58+25*3+(19-9)**2); -- 给变量赋值
    if v_result > 10 then
    DBMS_OUTPUT.PUT_LINE('平方根='||v_result); -- v_result 结果大于10，执行6-7行的语句
    DBMS_OUTPUT.PUT_LINE('结果为true，输出结果');
    end if; -- 结果为大于10，为true，执行到这里结束
    DBMS_OUTPUT.PUT_LINE('结果为fase，不满足条件'); -- if的条件结果不大于10，直接执行此语句，结束运行
end;
```

#### if……else

- 二选一的结果，要么显示if后的内容，要么显示else后的内容

```sql
if condition then -- condition 结果为true 执行if 后的 statements
statements;
else -- condition 结果为 false 执行 else 后的 statements
statements;
end if;
```

```sql
declare
    n_result number(10);
    begin
    n_result := 30;
    if n_result >= 30 then
        DBMS_OUTPUT.PUT_LINE('结果大于30'); -- 变量大于等于30执行这个
    else
        DBMS_OUTPUT.PUT_LINE('结果小于30'); -- 变量小于30执行这个
    end if;
end;
```

#### if……elsif

- 执行所有结果为 `true` 的语句，结果为 `false` 的语句就跳过，执行下一条语句，直到遇到 `end if`

```sql
declare
    n_student number(10);
begin
    n_student := DBMS_RANDOM.VALUE(100, 300);
    if n_student >= 100 and n_student <= 200 then
        DBMS_OUTPUT.PUT_LINE('数值范围在100~200之间');
    elsif n_student >= 200 and n_student <= 250 then
        DBMS_OUTPUT.PUT_LINE('数值范围在200~250之间');
    elsif n_student >= 250 and n_student <= 300 then
        DBMS_OUTPUT.PUT_LINE('数值范围在250~300之间');
        DBMS_OUTPUT.PUT_LINE('数值是' || n_student);
    else
        DBMS_OUTPUT.PUT_LINE('所有条件都不符合');
    end if;
end ;
```

#### if 嵌套

```sql
declare
    v_product products%rowtype;
    begin
    select * into v_product from PRODUCTS where PRODUCT_ID = 228;
    if v_product.LIST_PRICE>=2000 then -- 第一次判断成立，执行后面的语句
        DBMS_OUTPUT.PUT_LINE('该产品价格偏高');
        if v_product.STANDARD_COST>=2000 then -- 判断成立，结束此次判断，跳到 elsif 语句
            DBMS_OUTPUT.PUT_LINE('大于2000，不缺货');
            else
            DBMS_OUTPUT.PUT_LINE('小于2000，缺货');
        end if;
        -- 不管上面的if语句是否成立，都会执行这一句，这里的判断不成立，结束此次判断，之后的 if 和 else  都不会执行
        elsif v_product.LIST_PRICE >= 1000 and v_product.LIST_PRICE<=1500 then 
        DBMS_OUTPUT.PUT_LINE('中等价格');
        if v_product.STANDARD_COST >= 2000 then
            DBMS_OUTPUT.PUT_LINE('大于2000，不缺货');
            else
            DBMS_OUTPUT.PUT_LINE('小于2000，缺货');
        end if;
        else
        DBMS_OUTPUT.PUT_LINE('该产品价格偏低');
        if v_product.STANDARD_COST>=500 then
            DBMS_OUTPUT.PUT_LINE('不缺货');
            else
            DBMS_OUTPUT.PUT_LINE('缺货');
        end if;
    end if;
end;
```

#### 简单的 case 语句

- 给出一个表达式，并把表达式的结果同提供的几个可预见的结果比较，比较成功， 则执行对应的语句序列

```sql
[<< tag_name>>] -- 标签，可选；如果添加了，在代码结束处也需要加上这个标签；使用该便签可以提高代码可读性
case case_operand -- case_operand 变量，通常是一个表达式
when when_operand then -- when_operand 对应的是 case_operand 的结果；case 语句中，包含一个或多个 when ... then 语句
statement -- 如果 case_operand 和 when_operand 的结果相同，执行此 statement
[
  when when_operand then
  statement
]...
[else statement[statement]]..; -- 当所有的when_operand 都不能和 case_operand 对应，就会执行这个 else 语句
-- 如果没有ELSE子句，系统会给出CASE_NOT_FOUND异常
end case[tag_name]
```

```sql
declare
    v_categoryid varchar2(12);
begin
    select CATEGORY_ID into v_categoryid from PRODUCTS where PRODUCT_ID = 228;
    case v_categoryid
        when '2' then DBMS_OUTPUT.PUT_LINE(v_categoryid || '对应fake'); -- 找到对应的则会直接结束 case ，不会执行之后的 when...then 
        when '3' then DBMS_OUTPUT.PUT_LINE(v_categoryid || '对应aa');
        when '4' then DBMS_OUTPUT.PUT_LINE(v_categoryid || '对应bb');
        when '1' then DBMS_OUTPUT.PUT_LINE(v_categoryid || '对应ironman');
        else DBMS_OUTPUT.PUT_LINE('没有找到对应的类型'); 
        end case;
end;
```



#### 搜索式 case 语句

- 搜索式的CASE语句会依次检查布尔值是否为TRUE，一旦为TRUE，那么它所在的WHEN子句会被执行，而且它后面的布尔表达式将不被考虑。如果所有的布尔表达式都不为TRUE，那么程序将转到ELSE子句，如果没有ELSE子句，系统会给出CASE_NOT_FOUND异常

```sql
[<<tag_name>>]
case
when boolean_expression then statement;
[when boolean_expression then statement;]...
[else statement[statement]]..
end case <tag_name>
```

```sql
declare
    v_productinfo number(10,2); -- 声明变量
    begin
    select LIST_PRICE into v_productinfo from PRODUCTS where PRODUCT_ID = 98; -- 给变量赋值
    case
        when v_productinfo > 3000 then
        DBMS_OUTPUT.PUT_LINE('高价产品，价格是：'|| v_productinfo);
        when v_productinfo >= 2000 and v_productinfo <= 3000 then
        DBMS_OUTPUT.PUT_LINE('中价产品，价格是：'|| v_productinfo); -- 遇到为 true 的，结束此 case
        when v_productinfo <= 1000 then
        DBMS_OUTPUT.PUT_LINE('低价产品，价格是：'|| v_productinfo);
        else
        DBMS_OUTPUT.PUT_LINE('都不符合，价格是：'||v_productinfo);
        end case;
end;
```

#### loop 循环

```sql
[<<tag_name>>]
loop
statement,,
end loop
<tag_name>
```

##### 基本的loop语句

- 需要和条件控制语句一起使用，否则会死循环
- 使用 if 与 exit 的组合来终止循环；exit 必须在循环体内，它可以使循环无条件终止，并且可以继续执行循环体外的语句；如果和 if 搭配使用，则满足某个条件才会跳出循环，继续向下执行

```sql
declare
    n_num number(8) := 1; -- 定义变量并赋值
begin
    <<basicloop>>
    loop -- 进入循环体
        DBMS_OUTPUT.PUT_LINE('当前变量的值是：' || n_num);
        n_num := n_num + 1; -- 先讲变量加一，
        if n_num > 5 then -- 2不大于5，不执行接下来的语句，返回循环体的第一行重新执行
            DBMS_OUTPUT.PUT_LINE('结束，当前变量的值是：' || n_num); -- 变量大于5时，执行此语句
            exit basicloop; -- 终止循环，后面跟的是标签，增强可读性
        end if;
    end loop;
    DBMS_OUTPUT.PUT_LINE('循环结束');
end ;
```

- Exit 默认终止退出当前循环，但如果使用标签，可以终止并退出指定的`loop`循环
- 使用 `exit...when`来结束循环，游标常用；当 `when`后面的结果为`true`时，`exit`则会被触发，终止退出指定的循环;
- `exit`后不加 `loop`标签，则表示终止退出当前循环，可以替换简单的 `if`语句

```sql
declare
    n_num number(8) := 1;
begin
    <<basicloop>>
    loop
        DBMS_OUTPUT.PUT_LINE('当前变量的值是：' || n_num);
        n_num := n_num + 1;
            exit basicloop when n_num > 5; 
    end loop;
    DBMS_OUTPUT.PUT_LINE('循环结束');
end ;
```

##### while...loop

- 此机构本身可以终止`loop`循环，当`while`后的语句为 `true`时，`loop` 和`end loop`之间的语句集将执行一次，然后再次判断 `while` 后的表达式是否为 `true`

```sql
<<tag_name>>
while boolean_expression
loop
statement
end loop tag_name;
```

```sql
declare
    n_num number(8) :=1;
    begin
    DBMS_OUTPUT.PUT_LINE('当前变量的值是：'||n_num);
    <<while_loop>>
    while n_num<20
    loop
        if mod(n_num,3)=0 then
            DBMS_OUTPUT.PUT_LINE(n_num||'');
        end if;
        n_num := n_num+1;
        end loop;
    DBMS_OUTPUT.PUT_LINE('退出，当前变量的值是：'||n_num);
    DBMS_OUTPUT.PUT_LINE('结束循环');
end;
```

