---
title: Oracle SQL
date: 2017-07-30 21:31:43
categories:
    - 后端
tags: 
    - OracleDB
---

## Oracle 服务器配置

### Oracle 服务器实例

一个实例只能用于访问一个数据库

由内存和后台进程组成

由SID标识

Oracle的内存结构主要组成：

<img src="内存结构.PNG"></img>

专用进程实例：一个服务器进程对应一个用户进程

多进程实例一个服务器进程对应多个用户进程(轮流服务)

### 安装11G

1. 创建和配置数据库
2. 服务器类
3. 典型安装：配置安装位置与密码


### 初始数据库用户

username:Sys

username:System
password:安装时设置

### 登录数据库

系统用户：

1. sys,system(超级管理员)默认密码:安装时创建
2. sysman(用于企业管理)默认密码:安装时创建
3. scott(管理员)默认密码:tiger

启动sqlplus应用程序(稍后登录)

```
sqlplus /nolog
```

用户登录命令

```
conn username/password@连接字符串 as 身份(sysdba/sysoper/normal(默认))
```

以Sys身份登录()
1. 若sqlplus所运行的操作系统的登录身份在ora_dba组内的话不需要密码
>conn sys as sysdba
或
>conn / as sysdba

2. 若sqlplus所运行的操作系统的登录身份不在ora_dba组内
>conn sys/安装时设置的密码 as sysdba

锁定/解锁用户：
```sql
alter user 用户名 account lock/unlock;
```

物理数据库存储在：

程序安装目录\app\admin\oradata\数据库名`D:\app\admin\oradata\orcl`

### 连接远程服务器

网络服务的配置(相当于保存连接，会保存用户名和密码)：

需要：IP，端口，服务名，用户名和密码。(还需要开启监听程序)

>`conn system/root@10.1.59.26:1521/orcl`;

### 本地网络服务名配置

1. 打开`Net Configuration Assistant`
2. 本地网络服务名配置
3. 添加
4. 输入要连接的数据库名
5. TCP
6. 主机名输入要连接的主机的IP地址
7. 测试成功后
8. 输入数据库显示在本地的名称，例如：db
9. 使用该名称登录数据库：`conn system/root@db`

### 配置管理数据库

创建数据库：
1. 打开`Database Configuration Assistant`(简称DBCA)
2. 创建数据库
3. 选择模板：事务处理
4. 输入全局数据库名，访问时使用SID访问该数据库
5. 设置各个用户密码

--------------------------------------------------------------


### 查询所有表信息

dba_tables:数据库中的所有的表对象

user_tables:当前登录的用户所拥有的表对象

```sql
--example:
select table_name,tablespace_name from user_tables;
```

all_tables:当前登录的用户有权访问的表对象

```sql
--example:
select owner,table_name,tablespace_name from all_tables;
```

--------------------------------------------------------------

## 数据块体系结构
 
### 数据库开启顺序

shutdown->(读参数文件init.ora)nomount->(读控制文件)mount->(读所有文件)open

```sql
startup mount; -- 进入读控制文件状态
alter database open; -- 进入读所有文件状态
```
--------------------------------------------------------------

### 表空间

一个表会在一个表空间里以离散的数据块的形式存储。组成一张表的全部数据块称为一个segment(数据段)。事实上Oracle为了保证性能会将一张表里的数据块已连续的形式进行存储，每一个连续的数据存储空间成为一个数据区。

<img src="Oracle数据存储结构.PNG"></img>

创建表空间：

```sql
create tablespace 空间名 datafile 'd:/空间名.dbf' size 10m autoextend on next 10m maxsize 1g;

create tablespace table0725 datafile 'd:/0725.dbf' size 10m autoextend on next 10m maxsize 500m;

--example:
create tablespace shoppingtp datafile 'd:/shoppingtp01.dbf' size 10m autoextend on next 10m maxsize 1g;
alter tablespace shoppingtp add datafile 'd:/shoppingtp02.dbf' size 10m autoextend on next 10m maxsize 1g;
```

添加文件到表空间(扩大表空间)：
```sql
alter tablespace tptest add datafile 'd:/tptest02.dbf' size 10m;
```

查询表空间:

```sql
select name from v$tablespace;

-- 解释表dba_data_files
desc dba_data_files;

-- 查看SHOPPINGTP表空间的信息
select file_name,tablespace_name,bytes,blocks from dba_data_files where tablespace_name='SHOPPINGTP';

```

删除表空间：

```sql
drop tablespace xxx;
--若表空间非空
drop tablespace xxx including contents and datafiles;

--example:
drop tablespace table0725 including contents and datafiles;
```

查看表空间资源分配策略：
```sql
show parameter resource_limit
```

创建表要解决的问题: where(表存在哪),whose(谁能操作表)

--------------------------------------------------------------

## 用户权限管理

数据库模型：
1. 层次模型
2. 网络模型
3. 关系模型
4. 面向对象的数据模型

### 用户

验证方式：数据库验证、操作系统验证。

```sql
desc dba_users;

select username,password,created,default_tablespace from dba_users;

select username,created from dba_users;

-- 查询用户默认表空间
select username,default_tablespace from dba_users;
```
创建用户：

```sql
-- 密码验证abc
create user test_user identified by abc;

-- 创建一个默认表空间为tptest的用户且其在表空间上的配额为100M,密码默认为过期
create user test_user2 identified by abc default tablespace tptest quota 100m on tptest password expire;

--example:
create user shopping_user identified by abc default tablespace shoppingtp quota unlimited on shoppingtp;
```

修改用户：

```sql
-- 修改用户默认表空间
alter user test_user default tablespace table0725;
```

删除用户：

```sql
drop user test_user;
```

查询用户:

```sql
select * from all_users;
```

--------------------------------------------------------------

### 权限

1. 系统权限(system privelege)
2. 对象权限(object privelege)

select, update, delete, inseret, execute, index, reference, alter, read

赋予权限：

```sql
-- 将create session权限(可以登录)赋予test_user
grant create session to test_user;

-- 将create table权限(可以建表)赋予test_user
grant create table to test_user;

-- 将查询表t4的权限赋予test_user
grant select on t4 to test_user;
```

回收权限：

```sql
-- 将insert t4的权限从test_user回收
revoke insert on t4 from test_user;
```

访问其他用户创建的表需要以下操作：

```sql
-- t4表由test_user2创建
-- 使用test_user来访问t4表
-- test_user2与test_user同级

-- 将查询表t4的权限赋予test_user
grant select on t4 to test_user;
-- 将插入表t4的权限赋予test_user
grant insert on t4 to test_user;

select * from test_user2.t4;
```

### 角色

```sql
-- 将connect角色与resource角色赋予test_user
grant connect,resource to test_user2;
```

--------------------------------------------------------------

### 创建概要文件

```sql
-- 
show parameter resource_limit;
alter system set resource_limit = TRUE;
```

创建概要文件

```sql
create profile pro_test limit
sessions_per_user 3 -- 每个用户的会话允许个数
connect_time 3 -- 该用户每个会话的存活时间(分钟)
failed_login_attempts 2 -- 密码错误次数
;

create profile pro_test limit
sessions_per_user 3
connect_time 3
failed_login_attempts 2
;
```

查看用户对应的概要文件

```sql
select username,profile from dba_users;

-- 修改用户test_user的概要文件为pro_test
alter user test_user profile pro_test; 
```

--------------------------------------------------------------

## 表对象

关系型数据库中的完整性约束：
1. 实体完整性：每个表都要有主键(primary key)
2. 域完整性：每个数据项都要符合其对应的数据类型与长度等约束
3. 引用完整性：外键
4. 自定义完整性：check给数据项制定合法规则

创建表：

```sql
--example:
create table t_orders(
oid char(8) primary key, -- 主键
uiid char(6) references t_user(uiid), -- 外键(列级约束)
uiid char(6) constraint fk_uiid reference t_user(uiid),-- 有约束名的外键
foreign key(uiid2) constraint fk_uiid2 reference t_user(uiid)
, -- 表级有约束名的外键
customer varchar2(20) not null, -- 非空只能做列级约束
odate date default sysdate, -- 默认当前系统时间
girlfriend char(8) unique, -- 唯一，不能重复
onum number(2) check(onum>0), -- check实现自定义约束
);
```

```sql
--example:成绩表
--5W(学生数)*10(门课)=500000*1K(每条大小)=500M
--4(个学期)*500M=2000M=2G
create table cs(
--省略
)
pctfree 10 --当表空间剩余空闲空间<10%时停止插入操作
pctused 40 --当停止插入操作后表空间使用<40%时恢复插入操作
STORAGE( -- 存储参数设置
INTIAL 4096M -- 初始化导入数据大小
NEXT 500M -- 空间不够增加多少
)
```

将已有的表另存为新表(不会复制外键约束)：

```sql
create table emp as select * from employee;
```

删除数据：

```sql
-- 删除数据，可带条件，可恢复(速度慢)
delete * from emp;

-- 截断：不可恢复删除数据(速度快)，保留表结构
truncate table emp;

-- 删除数据和表，不可恢复
drop table emp;
```


--------------------------------------------------------------

## 分区

### 范围(RANGE)分区

例子：

根据商品价格分区

根据订单金额分区

```sql
/*3商品信息表*/
create table T_GOODS(
GID CHAR(6) PRIMARY KEY,
GNAME VARCHAR2(20) NOT NULL,
GTID CHAR(6) references T_TYPE(GTID),
GPRICE NUMBER(12,3) check(GPRICE>0),
GDISCOUNT NUMBER(5,2) DEFAULT 1,
GSTOCKS NUMBER(7,2),
GDATE DATE DEFAULT sysdate,
GMINSTOCKS NUMBER(7,2) check(GMINSTOCKS>0),
GMEMO VARCHAR2(50)
)TABLESPACE shoppingtp
pctfree 10 --商品信息表更新比较频繁
pctused 40 --说明
STORAGE ( 
   INITIAL 2G--说明
   NEXT 500M--说明
)
partition by range(GPRICE) -- 根据商品价格来分区
(partition part01 values less then
(100) tp01,
partition part02 values less then
(200) tp02,
partition part03 values less then
(300) tp03,
partition part04 values less then
(maxvalue) tp04
);
```

### 列表(LIST)分区

对可枚举类型进行分区

例子：

根据商品类型分区

根据性别分区

```sql
/*7订单主表*/
create table T_MAIN_ORDER(
OMID CHAR(12) PRIMARY KEY,
UIID CHAR(6) references T_USER(UIID),
ODATE DATE DEFAULT sysdate,
OAMOUNT NUMBER(12,3),
OSTATE CHAR(1) check(PSTATE = 1 or PSTATE = 2 or PSTATE = 3 or PSTATE = 4),
PMEMO VARCHAR2(50)
)TABLESPACE shoppingtp
pctfree 10 --说明
pctused 40 --说明
STORAGE ( 
   INITIAL 1G--说明
   NEXT 500M--说明
)
-- 假设数据分析之后

-- 第一种情况
-- PSTATE = 1占10%
-- PSTATE = 2占10%
-- PSTATE = 3占30%
-- PSTATE = 4占40%
-- 建议1和2分一个区
-- 3分一个区
partition by list(OSTATE)
(
partition part01 values('1','2'),
partition part02 values('3'),
partition part03 values('4')
)

-- 第二种情况
-- 经常查询1的频率80%
-- 2的频率10%
-- 3的频率10%
-- 4基本不查询
partition by list(OSTATE)
(
partition part01 values('1'),
partition part02 values('3','2'),
partition part03 values('4')
);
```

### HASH分区

例子：

根据商品编号分区

根据订单编号分区

```sql
/*1注册用户表*/
create table T_USER(
UIID CHAR(6) PRIMARY KEY,
UNAME VARCHAR2(20) NOT NULL,
UBIRTHDAY DATE,
USEX CHAR(1) check(USEX ='F' or USEX ='M'),
UADDRESS VARCHAR2(50),
UADDRESS VARCHAR2(20)
)TABLESPACE shoppingtp
pctfree 10 --说明
pctused 40 --说明
STORAGE ( 
   INITIAL 1G--用户可能
   NEXT 100M--说明
)
partition by hash(UIID)
(
partition part01 tablespace shoppingtp01,
partition part02 tablespace shoppingtp02
);
```

--------------------------------------------------------------

## 锁

Oracle会给修改、插入的数据加一个行级锁，一次只允许一个用户修改数据，若此时还有其他用户在进行查询会返回修改前的版本。

1. 当用户对数据进行insert/update/delete操作的时候回自动加上行级锁，失误结束的时候解锁
2. select ... from ... for update加上行级锁，事务结束的时候解锁
3. 可以使用lock table语句对表进行表级锁
  - 行共享 (ROW SHARE) – 禁止排他锁定表
  - 行排他(ROW EXCLUSIVE) – 禁止使用排他锁和共享锁
  - 共享锁(SHARE)锁定表，仅允许其他用户查询表中的行禁止其他用户插入、更新和删除>行多个用户可以同时在同一个表上应用此锁
  - 共享行排他(SHARE ROW EXCLUSIVE) – 比共享锁更多的限制，禁止使用共享锁及更高的锁
  - 排他(EXCLUSIVE) – 限制最强的表锁，仅允许其他用户查询该表的行。禁止修改和锁定表
4. 死锁：Oracle会自动检测并智能结束其中一个锁

```sql
-- 共享锁：其他用户只能查询，不可修改
lock table t_studentuser in share mod;


```

--------------------------------------------------------------

## **`SQL`**

### 集合查询

并集

```sql
select e1.*,'经理为King'
from employees e1 join employees e2
on e1.manager_id=e2.employee_id
where e2.last_name='King'
union all
select employees.*,'高工资'
from employees
where salary>10000
union all
select employees.*,'部门为1700'
from employees
where department_id in (
    select department_id
    from departments
    where location_id=1700
);
```

### 视图

```sql
-- 创建视图
create view v_90 as
select * from employees where department_id=90;

-- 查询视图
select * from v_90;

-- 插入视图
insert into v_90
values(...);


```

### 动态 `SQL`

使用execute immediate语句来执行位于字符串中的SQL语句

使用动态SQL创建用户需要sys权限

```sql
create or replace procedure p_createstudentuser
as
cursor cur_userid is
    select userid
        from system.t_studentuser;

v_userid system.t_studentuser.userid%type;
begin
    open cur_userid;
    fetch cur_userid into v_userid;
    while (cur_userid%found) loop
        execute immediate 'create user '||v_userid' identified by abc';
        execute immediate 'grant connect,resource,create view to '||v_userid;
        fetch cur_userid into v_userid;
    end loop;
    close cur_userid;
end;
```

--------------------------------------------------------------

## **Tips**

### 使用序列来代替需要自增的属性

```sql
-- 创建序列
-- 序列名seq_empid
-- 从100开始自增，每次自增2
create sequence seq_empid
start with 100
increment by 2;

-- 使用序列
-- 使用nextval来获取下一个值
-- 使用currval来获取当前值
insert into emp
values(seq_empid.nextval,'辑','罗','12345678@qq.com','123456789',to_date('2017-07-26','yyyy-mm-dd'),'FI_MGR',50000,0.1,null,60);
```

```sql
-- nvl(commission_pct,0)若commission_pct属性为空则设为0
-- 若commission_pct属性的值为空则设为0再+0.1
update employee set commission_pct=nvl(commission_pct,0)+0.1
```

### 使用虚拟表dual来进行查询

```sql
-- 查询当前系统时间
-- 注意：sysdate不是dual的一个属性，而是获取当前系统时间的函数
select sysdate from dual; 
```

### 以指定格式获取当前日期

```sql
select to_char(sysdate,'yyyy-mm-dd') from dual;
select to_char(sysdate,'yyyy"年"mm"月"dd"日"') from dual;
select to_char(sysdate,'yyyy-mm-dd hh:mi:ss') from dual;
```

### 日期计算

```sql
-- 在获取30天后的日期
select sysdate+30 from dual;

-- 计算相差天数
select trunc(sysdate-to_date('2014-09-07', 'yyyy-mm-dd')) 入学天数 from dual;

-- 计算相差月份数
select trunc(months_between(sysdate-to_date('2014-09-07', 'yyyy-mm-dd'))) 入学月份 from dual;
```

### 绑定变量

```sql
select * from employees
where manager_id=&经理编号;

select * from employees
where first_name like '%&名称%';
```

### 显示指定条数的数据

在查询结果后系统会自动增加一个rownum行号

```sql
select *
from(select * from employees)
where rownum<10;

select *
from(
    select * from employees
    order by salary desc
)
where rownum<10;
```

### null处理函数nvl2(被判断的属性,若不为空的值,若为空的值)


### null处理函数coalesece()

```sql
-- 以此从参数里面找非空的作为值
select *
coalesece(ifmanager,ifbase,iftemp) jobid,
coalesece(msal,bsal,tsal) sal
from test;
```

### 函数decode()

decode(temp,'1','经理'，'2','业务员','3','临时工','其他')

若temp的值为1则整个值为经理

### 如何将Excel表数据导入Oracle数据库

步骤：
1. 将excel另存为由制表符分割的txt
2. 创建一个控制文件(input.ctl)，如下：
  - 导入
  - 导入文件地址
  - 以追加模式(append或insert,replace替换)导入表t_studentuser
  - 字段分割符为'09'即制表符
  - 表的属性

```ctl
load data
infile 'd:\student.txt'
append into table t_studentuser
fields terminated by X'09'
(userid,manager,emp01,emp02,emp03)
```

3. 在cmd中使用sqlldr工具导入，命令如下

```cmd
cmd>sqlldr userid=system/abc123 control='d:\input.ctl'
```

--------------------------------------------------------------

## **Q & A**

Q: `QRA-12560：TNS 协议适配器错误`

A: OracleServiceORCL服务没有开启,cmd 中输入`net stop oracleserviceorcl` 来开启

Q: `QRA-28000：the account is locked`

A: 用户已被锁定。需要使用该用户上一级用户执行以下命令来解锁。命令如下：`alter user 被锁定的用户名 account unlock;`

Q: 修改密码

A: 
1. 修改其它账户的密码： 命令：`alter user system identified by abc 123;` system 为需要修改的用户名， abc 123 为新密码
2. 修改本账户密码：登陆后,输入 `password` 输入新密码即可。

Q: 监听程序无法识别连接描述符中请求服务

A: 

Q: 无监听程序

A: 没开监听服务。配置监听程序,打开程序`Net Configuration Assistant`进行配置

Q: 删除了某个表空间的数据文件之后无法打开数据库
>- ORA-01109数据库未打开
>- ORA-01157无法标识/锁定数据文件

A: 删除出问题的数据文件:`Alter database ‘d: tpshopping01.dbf drop offline`

Q: ORA-00942 表或视图不存在

A: 真的不存在或查询的用户权限不够。

Q: ORA-01549 表空间非空，请使用including contents选项

A: 需要使用以下命令来删除表空间(同时会删除表空间里的数据与文件)`drop tablespace table0725 including contents and datafiles;`

Q: 设置日期格式

A: Oracle默认日期格式为：'26-7月 -17'，可使用以下代码来替换Oracle格式日期，to_date('2017-07-26','yyyy-mm-dd')

Q: 删除有外键的数据

A: 监听器位置：D:\app\admin\product\11.2.0\dbhome_1\NETWORK\ADMIN
