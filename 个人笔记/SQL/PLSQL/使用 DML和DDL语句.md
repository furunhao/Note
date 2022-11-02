- DML语句的使用
```sql
DECLARE  
    v_catgid VARCHAR2(10) := 0;  
    v_bol    BOOLEAN      := TRUE;  
BEGIN  
    SELECT CATEGORYID  
    INTO v_catgid  
    FROM CATEGORYINFO  
    WHERE CATEGORYNAME = '电脑';  
    DBMS_OUTPUT.PUT_LINE('电脑对应的编码存在：' || v_catgid);  
  
EXCEPTION  -- 上面的不符合条件会执行此语句
    WHEN NO_DATA_FOUND THEN  -- 预定义异常
        IF v_bol THEN  
            DBMS_OUTPUT.PUT_LINE('没有对应的编码，为其增加该产品类型');  
            INSERT INTO CATEGORYINFO VALUES ('0100000001', '电脑'); 
            -- 没有对应记录时，会增加一条记录，只会在触发 NO_DATA_FOUND 时才会执行 
            COMMIT; -- 提交事务       
            END IF;    
            WHEN TOO_MANY_ROWS THEN  -- 假如最上面的语句返回多条记录时，会执行此语句
        DBMS_OUTPUT.PUT_LINE('对应数据过多，请确认！');  
END;
```
- DDL语句的使用
```sql
declare
    table_name varchar2(200);
    begin
    table_name := 'create table test_DDL(
        id varchar2(10),
        name varchar2(10)
        )';
    execute immediate table_name; -- 利用命令创建表
    -- 利用 execute immediate 执行 DDL 语句在存储过程常用此方式，提高代码可移植性
end;
```

