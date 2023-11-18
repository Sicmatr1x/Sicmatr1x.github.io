---
title: Oracle PL
date: 2017-07-30 21:31:43
categories:
    - Database
tags: 
    - Oracle
    - Back-end
---

## 基础概念

特点：

1. 模块化
2. 过程化
3. 提高操作性能，减少网络传输
4. 错误处理

分类：

1. 匿名块
2. 存储过程(procedure)
3. 函数(function)
4. 包(package)
5. 触发器(trigger)

## 编写 PL/SQL

### 函数

PL/SQL由三部分组成：

1. 定义部分
2. 可执行部分
3. 例外处理部分

```sql
-- 定义部分
declare

-- 可执行部分
Begin

-- 例外处理部分
exception

End;
```

```sql
--example:
select first_name, email,salary
from employees
where employee_id=&员工编号;
-- 改写成为程序：
set serveroutput on -- 在本次会话中开启系统输出显示
-- 定义部分
declare
    v_name varchar2(20);
    v_email varchar2(25);
    v_sal number(8,2);
begin-- 可执行部分
    select first_name, email,salary
    into v_name, v_email, v_sal
    from employees
    where employee_id=&员工编号;
    dbms_output.put_line('姓名'||v_name||'邮箱'||v_email||'工资'||v_sal);
exception -- 例外处理部分(异常处理)
    when no_data_found then
        dbms_output.put_line('您输入的工号不存在');
    when others then
        dbms_output.put_line(sqlerrm);--sqlerrm是系统显示异常的变量
end;
```

#### 无参数返回值函数样例

```sql
-- 计算个人所得税
declare
    v_salary number(8,2):=&工资;
    v_shouldTaxedNum number(8,2);
    v_taxed number(8,2);
begin
    v_shouldTaxedNum := v_salary - 3500;
    if v_shouldTaxedNum <= 0 then
        v_taxed := 0;
    elsif v_shouldTaxedNum <= 1500 then
        v_taxed := v_shouldTaxedNum*0.03-0;
    elsif v_shouldTaxedNum <= 4500 then
        v_taxed := v_shouldTaxedNum*0.1-105;
    elsif v_shouldTaxedNum <= 9000 then
        v_taxed := v_shouldTaxedNum*0.2-555;
    elsif v_shouldTaxedNum <= 35000 then
        v_taxed := v_shouldTaxedNum*0.25-1005;
    elsif v_shouldTaxedNum <= 55000 then
        v_taxed := v_shouldTaxedNum*0.3-2755;
    elsif v_shouldTaxedNum <= 80000 then
        v_taxed := v_shouldTaxedNum*0.35-5505;
    else
        v_taxed := v_shouldTaxedNum*0.45-13505;
    end if;
    dbms_output.put_line('v_taxed='||v_taxed);
end;
```

#### 有参数返回值函数样例

```sql
create or replace function f_tax(v_salary in number)
return number
as
    v_shouldTaxedNum number(8,2);
    v_taxed number(8,2);
begin
    v_shouldTaxedNum := v_salary - 3500;
    if v_shouldTaxedNum <= 0 then
        v_taxed := 0;
    elsif v_shouldTaxedNum <= 1500 then
        v_taxed := v_shouldTaxedNum*0.03-0;
    elsif v_shouldTaxedNum <= 4500 then
        v_taxed := v_shouldTaxedNum*0.1-105;
    elsif v_shouldTaxedNum <= 9000 then
        v_taxed := v_shouldTaxedNum*0.2-555;
    elsif v_shouldTaxedNum <= 35000 then
        v_taxed := v_shouldTaxedNum*0.25-1005;
    elsif v_shouldTaxedNum <= 55000 then
        v_taxed := v_shouldTaxedNum*0.3-2755;
    elsif v_shouldTaxedNum <= 80000 then
        v_taxed := v_shouldTaxedNum*0.35-5505;
    else
        v_taxed := v_shouldTaxedNum*0.45-13505;
    end if;
    return v_taxed;
end;
/
show error;/*如果编译出现错误则显示错误信息*/

-- 在查询语句中调用函数
select employee_id, nvl(salary,0)+nvl(salary,0)*nvl(commission_pct,0) sal, f_tax(nvl(salary,0)+nvl(salary,0)*nvl(commission_pct,0)) tax
from employees;
```

#### 规范函数样例

```sql
/*
函数名称：f_returngoodprice
函数参数说明：输入参数：good_id：字符串
函数返回值：该商品的销售单价：数值
创建人：
创建时间：
修改人：
修改时间：
*/
create or replace function f_returngoodprice(good_id varchar2)
return number as
v_goodprice t_goods.gprice%type;--锚定变量类型
begin
    select nvl(gprice,0)*nvl(gdiscount,1)
        into v_goodprice
        from t_goods
            where upper(trim(gid))=upper(trim(good_id));
    return v_goodprice;
end f_returngoodprice;
/
show error;
```

```sql
/*
函数名称：f_returngmaxstocks
函数参数说明：输入参数：good_id：字符串
函数返回值：该商品的最大库存：数值
创建人：
创建时间：
修改人：
修改时间：
*/
create or replace function f_returngmaxstocks(good_id varchar2)
return number as
v_gmaxstocks t_goods.gmaxstocks%type;
begin
    select nvl(gmaxstocks,0)
        into v_gmaxstocks
        from t_goods
            where upper(trim(gid))=upper(trim(good_id));
    return v_gmaxstocks;
end f_returngmaxstocks;
/
show error;
```

#### 自动主键编码

```sql
create sequence seq_uiid;

/*
函数名称：f_createuiid
函数参数说明：输入参数：
函数返回值：使用序列生产的uiid主键
创建人：
创建时间：
修改人：
修改时间：
*/
create or replace function f_createuiid return varcahr2
as
v_uiid varchar2(6);
v_seq number;
begin
    select seq_uiid.nextval into v_uiid from dual;
    v_uiid:=trim(to_char(v_seq,'000000'));
    return v_uiid;
end f_createuiid;
```

```sql
/*
函数名称：f_createpmid
函数参数说明：输入参数：
函数返回值：使用序列生产的pmid主键
创建人：
创建时间：
修改人：
修改时间：
*/
create or replace function f_createpmid return varchar2
as
v_pmid varchar2(12);
v_pmidin t_main_procure.pmid%type;
begin
    v_pmid:='P'||to_char(sysdate,'yyyymm');
    select max(pmid)
        into v_pmidin
        from t_main_procure
            where to_char(sysdate,'yyyymm')=to_char(pdate,'yyyymm');
    if v_pmidin is null then
        v_pmid:=v_pmid||'00001';
    else
        v_pmid:=v_pmid||trim(to_char(to_number(substr(v_pmidin,8,5))+1,'00000'));
    end if;
    return v_pmid;
end f_createpmid;
/
show error;

select f_createpmid() from dual;
```

```sql
/*
函数名称：f_returnisenoughgoods
函数参数说明：输入参数：good_id：字符串，num销售数量
函数返回值：该商品的剩余库存是否够卖：数值
创建人：
创建时间：
修改人：
修改时间：
*/
create or replace function f_returnisenoughgoods(good_id varchar2, num number)
return number as
v_gstocks t_goods.gstocks%type;
enough number(1,0):=-1;
begin
    select nvl(gstocks,0)
        into v_gstocks
        from t_goods
            where upper(trim(gid))=upper(trim(good_id));
    if v_gstocks-num>0 then
        enough:=1;
    end if;
    return enough;
exception
    when others then
        return -2;
end f_returnisenoughgoods;
/
show error;
select f_returnisenoughgoods('g0005',15) from dual;
```

#### 操作函数

显示用户有的对象：

```sql
select object_name,object_type
from user_objects;
```

查看函数代码：

```sql
desc user_source

select text
from user_source
where name='F_RETURNISENOUGHGOODS';
```

删除函数：

```sql
drop function F_RETURNISENOUGHGOODS;
```

### 存储过程

存储过程样例：
- 隐式游标：
- cursor的名称是：SQL：每执行一条SQL语句以后自动创建的私有内存空间
- SQL%found:布尔值，true:语句执行成功,false:失败
- SQL%rowcount:integer,insert,update,delete,select影响的行数


```sql
-- 增加一张采购单
/*
存储过程名称：p_insertmainprocure
输入参数：i_sid:供应商编号:i_memo:备注:default:test
输出参数:o_result:1:添加成功,-1:发生错误,0:添加失败,-2:供应商不存在
创建人：
创建时间：
*/
create or replace procedure p_insertmainprocure(i_sid in t_main_procure.sid%type, i_memo in t_main_procure.pmemo%type default 'test', o_result out number)
as
v_count number:=0;
begin
    select count(sid)
        into v_count
        from t_supplier
            where lower(trim(sid))=lower(trim(i_sid));
    if v_count=0 then
        o_result:=-2;
        return o_result;
    end if;

    insert into t_main_procure
        values(f_createpmid(), i_sid, sysdate, null, '1', i_memo);
    --这里的SQL为隐式游标，%用来提取属性(影响了多少行)
    if SQL%rowcount>0 then
        o_result:=1;--添加数据成功
    else
        o_result:=0;--添加数据失败
    end if;
    commit;
exception
    when others then
        o_result:=sqlcode;
end p_insertmainprocure;
```

```sql
-- 没有输出参数的存储过程可以这样调用
exec p_insertmainprocure();

-- 有输出参数的存储过程，只能在程序中调用
declare
v_result number;
begin
    p_insertmainprocure('p002','通过存储过程添加',v_result);
    if v_result=1 then
        dbms_output.put_line('添加成功');
    elsif v_result=-2 then
        dbms_output.put_line('添加失败');
    else
        dbms_output.put_line('error');
    end if;
end;
```

```sql
-- 4)设计一个函数，实现订单编号的自动编码：'Oyyyymm00XXX'（练习）
create or replace function f_createomid
return varchar2
as
    v_omid varchar2(12);
    v_maxomid t_main_order.omid%type;
begin
    v_omid:='O'||to_char(sysdate,'yyyymm');
    ---去订单查询当月最大的那张单
    select max(omid) into v_maxomid
        from t_main_order where to_char(odate,'yyyymm')=to_char(sysdate,'yyyymm');
    if v_maxomid is null then
        v_omid:=v_omid||'00001';--当月第一张单
    else
        v_omid:=v_omid||trim(to_char(to_number(substr(v_maxomid,8,5))+1,'00000'));
    end if;-- --当月下一张单
    return v_omid;
exception
    when others then
        return null;
end;
/
show error;

select f_createomid from dual;

添加订单明细数据:
/*
  存储过程名称:p_insertorderitems
  输入参数:i_omid:订单编号;i_gid :商品编号；i_onum：数量,默认值为1
  输出参数:o_result:1:添加成功,-1:发生错误，0：添加不成功,-2:输入数据不准确
  创建人:miss lee
  创建日期:
  修改:
*/
create or replace procedure p_insertorderitems(i_omid in t_order_items.omid%type, i_gid in t_order_items.gid%type,  i_onum in number default 1,o_result out number)
as
v_count number:=0;
begin
    select count(omid) into v_count      --检查订单编号是否正确
        from t_main_order where trim(omid)=trim(i_omid);
    if v_count<1 then
        o_result:=-2;
        return;  --退出程序
    end if;
    select count(gid) into v_count    --检查商品编号是否正确
        from t_goods where trim(gid)=trim(i_gid);
    if v_count<1 then
        o_result:=-2;
        return;  --退出程序
    end if;
    insert into t_order_items
        values(i_omid,i_gid,f_returngoodprice(i_gid),i_onum,null);
    o_result:=1;
    commit;
exception
    when others then
        o_result:=sqlcode;
        rollback;
 end;
/
show error;

declare
v_result number;
begin
    p_insertorderitems('O20170800001','g0006',10,o_result=>v_result);
    if v_result=1 then
        dbms_output.put_line('添加成功');
   elsif  v_result=-2 then
        dbms_output.put_line('数据不存在');
   else
        dbms_output.put_line(v_result);
   end if;
end;

create or replace function f_returngoodprice(good_id varchar2)
   return number as
v_goodprice t_goods.gprice%type;--锚定变量
begin
    select nvl(gprice,0)*nvl(gdiscount,1)
        into v_goodprice
        from t_goods
            where upper(trim(gid))=upper(good_id); ---
    return v_goodprice;
exception
    when others then
        return 0;--自定义一个错误返回值
end f_returngoodprice;

```

#### 操作返回结果为多条数据(列表)的情况

```sql
create or replace procedure p_printemp(i_deptid number)
as
v_name employees.first_name%type;
v_sal employees.salary%type;
v_hdate employees.hire_date%type;
-- 1.定义游标
cursor cur_emp is
    select first_name,salary,hire_date            
        from employees
            where department_id=i_deptid;
begin
    -- 2.打开游标
    open cur_emp;
    -- 3.提取数据
    fetch cur_emp into v_name,v_sal,v_hdate;
    if cur_emp%notfound then
        dbms_output.put_line('您输入的编号不存在');
    end if;
    -- 循环提取
    while(cur_emp%found) loop
        dbms_output.put_line(v_name||'.'||v_sal||'.'||v_hdate);
        fetch cur_emp into v_name,v_sal,v_hdate;
    end loop;

    --4.关闭游标
    close cur_emp;

exception
    when others then
        dbms_output.put_line(sqlerrm);
end p_printemp;
```

#### 操作返回结果为多条数据(列表)的情况返回任意想要的属性值

```sql
create or replace procedure p_printemp2(i_deptid number)
as
-- 1.定义游标
cursor cur_emp is
    select *
        from employees
            where department_id=i_deptid;
-- 游标的行记录类型
rec_emp cur_emp%rowtype;
begin
    -- 2.打开游标
    open cur_emp;
    -- 3.提取数据
    fetch cur_emp into rec_emp;
    if cur_emp%notfound then
        dbms_output.put_line('您输入的编号不存在');
    end if;
    -- 循环提取
    while(cur_emp%found) loop
        dbms_output.put_line(rec_emp.last_name||'.'||rec_emp.salary);
        fetch cur_emp into rec_emp;
    end loop;

    --4.关闭游标
    close cur_emp;

exception
    when others then
        dbms_output.put_line(sqlerrm);
end p_printemp2;
/
show error;
```

```sql
for v_i in 1...1000 loop

end loop;
```

```sql
create or replace procedure p_printdeptandemp
as
-- 1.定义游标
cursor cur_dept is
    select *
        from departments;
cursor cur_emp(deptno number) is
    select *
        from employees
            where department_id=deptno;
-- 游标的行记录类型
rec_dept cur_dept%rowtype;
rec_emp cur_emp%rowtype;
begin
    open cur_dept;
    fetch cur_dept into rec_dept;
    while(cur_dept%found) loop
        dbms_output.put_line('部门:'||'.'||rec_dept.department_id||'.'||rec_dept.department_name||'.'||rec_dept.location_id);

        for rec_emp in cur_emp(rec_dept.department_id) loop
            dbms_output.put_line('    员工:'||'.'||rec_emp.employee_id||'.'||rec_emp.first_name||'.'||rec_emp.salary);
        end loop;

        fetch cur_dept into rec_dept;
    end loop;

    --4.关闭游标
    close cur_dept;
end p_printdeptandemp;
/
show error;
```

#### 定义异常

```sql
declare
v_location number;
v_dept number;
-- 1.定义异常名
e_datanotnull exception;
-- 2.将已有的异常关联到异常名
pragma exception_init(e_datanotnull, -1400);

e_depttohigh exception;
begin
    v_location:=&部门编号
    v_dept:=&部门编号
    if v_dept>10000
        raise e_depttohigh;
    end if;
    if v_location>10000
        -- 给自定义异常编异常号码需在(-20000,-30000)区间内
        raise_application_error(-20001,'地址编号太大');
    end if;

exception
    when e_depttohigh then
        dbms_output.put_line('v_dept>10000');
    when others then
        dbms_output.put_line(sqlerrm);
end;
```


```sql
/*
  存储过程名称:p_checkorder
  功能描述：
  0.检查该单据是否已经审核，若审核则抛出自定义异常
  1.计算采购单的总金额
  2.更新采购明细的商品的库存、单价(加权平均法)
  3.更新采购单的状态为已审核

  输入参数:i_pmid采购单编号;
  输出参数:o_result:1:审核成功,0：发生未知错误
  异常：-20001:该单据状态不是待审核;-20002:有商品超出最高库存(裁剪);-20003:该单据不存在;-2004:该单据没有明细
  创建人:
  创建日期:
  修改:
*/
create or replace procedure p_checkorder(i_pmid T_MAIN_PROCURE.pmid%type, o_result out number)
as
v_count number:=0;
v_pstate T_MAIN_PROCURE.pstate%type;
v_pamount T_MAIN_PROCURE.pamount%type;
v_gstocks T_GOODS.gstocks%type;
v_gminstocks T_GOODS.gminstocks%type;
-- 若已审核则抛出异常
e_alreadychecked exception;
-- 1.定义游标
cursor cur_items is
    select *
        from T_PROCURE_ITEMS
            where lower(trim(pmid))=lower(trim(i_pmid));
-- 游标的行记录类型
rec_items cur_items%rowtype;
begin
    v_pamount:=0;

-- 判断采购单存不存在
    select count(pmid)
        into v_count
        from T_MAIN_PROCURE
            where lower(trim(pmid))=lower(trim(i_pmid));
    if v_count=0 then
        raise_application_error(-20003,'该单据不存在');
    end if;

-- 0.检查该单据是否已经审核，若审核则抛出自定义异常
    select pstate
        into v_pstate
        from T_MAIN_PROCURE
        where lower(trim(pmid))=lower(trim(i_pmid));
    if v_pstate<>1 then
        raise_application_error(-20001,'该单据状态不是待审核');
    end if;

--  1.计算采购单的总金额
--  2.更新采购明细的商品的库存、单价(加权平均法)
    open cur_items;

    fetch cur_items into rec_items;
    if cur_items%notfound then
        raise_application_error(-20004,'该单据没有明细');
    end if;
    while(cur_items%found) loop
        update t_goods
        set gstocks=gstocks+rec_items.pinum, gprice=round((gstocks*gprice+rec_items.pinum*rec_items.piprice
        )/(gstocks+rec_items.pinum))
        where gid=rec_items.gid;
        v_pamount:=v_pamount+rec_items.pimoney;
        fetch cur_items into rec_items;
    end loop;

    close cur_items;

    dbms_output.put_line('采购单'||i_pmid||'的总金额'||v_pamount);

-- 3.更新采购单的状态为已审核
    update T_MAIN_PROCURE set T_MAIN_PROCURE.pstate='2',T_MAIN_PROCURE.pamount=v_pamount
    where lower(trim(pmid))=lower(trim(i_pmid));
    
    o_result:=1;
    commit;

exception
    when others then
        o_result:=sqlcode;
        rollback;
end p_checkorder;
/
show error;
```

### 触发器 `trigger`

#### 行级触发器样例

>:new 代表：
>- insert操作中添加的行记录
>- update操作中更新以后的行记录

>:old 代表：
>- delete操作中删除的行记录
>- update操作中更新以前的行记录

若在触发器中想要阻止某个操作只能通过抛出一个异常来中断

```sql
create or replace trigger tr_checkpitemsinsert
before insert on t_procure_items
for each row --  行级触发器
declare
    v_state t_main_procure.pstate%type;
begin
    --  判断所添加的明细数据的单据是否是待审核
    --  if not 待审核
    select pstate
        into v_state
        from t_main_procure
            where pmid=:new.pmid;
    if v_state<>'1' then
        raise_application_error('-20001','该单据已经审核不能再添加');
    end if;
end;
/
show error;
```

####  触发器的分类

1. DML操作：insert,update,delete
2. 视图的替代触发器
3. DDL操作:create,alter
4. 数据库系统级别触发器

#### DML 触发器

timing:
- before:执行操作之前进行处理
- after:执行操作之后进行处理
- instead of:在视图上进行操作时进行替代

object_name:table or view的名字


```sql
create or replace trigger trigger_name
timing r_body on object_name
declare
    
begin

end;
```

#### 合并触发器

如果多个触发器是同一个表上，进行同一个时间介词(before,after)，同一个触发次数类型(语句级的，行级)则可合并触发器

```sql
create or replace trigger tr_checkpitems
before update or insert or delete on t_procure_items
for each row --  行级触发器
declare
    v_state t_main_procure.pstate%type;
begin
    if inserting then
        select pstate
            into v_state
            from t_main_procure
                where pmid=:new.pmid;
    elsif updating or deleting then
        select pstate
            into v_state
            from t_main_procure
                where pmid=:old.pmid;
    end if;

    if v_state<>'1' then
        raise_application_error('-20001','该单据已经审核不能再删除
');
    end if;
end;
/
show error;
```

```sql
create or replace trigger tr_updateprocuremainmoney
after update or delete of PPRICE,PNUM or insert on t_procure_items
for each row
declare
    v_pamount t_main_procure.PAMOUNT%type;
begin

    if inserting then
        update t_main_procure set
            pamount=nvl(pamount,0)+:new.pinum*:new.piprice;
                where pmid=:new.pmid;

    elsif updating then
        update t_main_procure set
            pamount=nvl(pamount,0)-:old.pinum*:old.piprice+:new.pinum*:new.piprice;
                where pmid=:new.pmid;

    elsif deleting then
        update t_main_procure set
            pamount=nvl(pamount,0)-:old.pinum*:old.piprice;
                where pmid=:new.pmid;

    end if;
   
end;
/
show error;
```

#### 日志触发器样例

```sql
create table log(
lid number(9,0) primary key,
employee_id number(6,0) references employees(employee_id),
old_salary number(8,2),
new_salary number(8,2),
opdate Date,
opuser varchar2(20)
);
create sequence seq_lid
start with 1
increment by 1;

create or replace trigger tr_updatemployeesalary
after update on employees
for each row
declare
    
begin
    insert into log values(seq_lid.nextval, :new.employee_id, :old.salary, :new.salary, sysdate, user);
end;
/
show error;

update employees set salary=20000
where employee_id=100;
```

#### 登录触发器

```sql
-- 记录用户的登录信息
create or replace trigger tr_logon
after logon on database
begin
    insert into t_mylog
    values(seq_log.nextval,user,default,'logon',sys_context('USERENV','IP_ADDRESS'));
end;
-- 记录用户的登出信息
create or replace trigger tr_logoff
before logoff on database
begin
    insert into t_mylog
    values(seq_log.nextval,user,default,'logoff',sys_context('USERENV','IP_ADDRESS'));
end;
```
