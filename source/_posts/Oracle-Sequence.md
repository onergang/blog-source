title: Oracle-Sequence
date: 2016-05-28 11:54:54

categories: 技术

tags: Oracle
---

最近在工作中频繁的用到了序列，以前没有注意过，现在抽个时间集中整理下。

<!--more-->
在oracle中sequence就是序号，每次取的时候它会自动增加。sequence与表没有关系。 

### Create Sequence

`首先要有CREATE SEQUENCE或者CREATE ANY SEQUENCE权限。`

创建语句如下： 

```plsql
  CREATE SEQUENCE seqTest
    INCREMENT BY 1 -- 每次加几个
    START WITH 1 -- 从1开始计数
    NOMAXvalue -- 不设置最大值
    NOCYCLE -- 一直累加，不循环
    CACHE 10; --设置缓存cache个序列，如果系统down掉了或者其它情况将会导致序列不连续，也可以设置为---------NOCACHE
```

### Get Sequence

定义好sequence后，你就可以用currVal，nextVal取得值。

`NextVal：增加sequence的值，然后返回 增加后sequence值`。

`CurrVal：返回 sequence的当前值,必须执行nextval之后才能执行这个。`

  得到值语句如下：
​        

```plsql
SELECT Sequence名称.CurrVal FROM DUAL; 
```

  如得到上边创建Sequence值的语句为：

```plsql
select seqtest.currval from dual
```

在Sql语句中可以使用sequence的地方： 

```plsql
   - 不包含子查询、snapshot、VIEW的 SELECT 语句 
   - INSERT语句的子查询中 
   - INSERT语句的values中 
   - UPDATE 的 SET中
```

如在插入语句中

```plsql
insert into 表名(id,name)values(seqtest.Nextval,'sequence 插入测试');
```

 注：
第一次NEXTVAL返回的是初始值；随后的NEXTVAL会自动增加你定义的INCREMENT BY值，然后返回增加后的值。
 CURRVAL 总是返回当前SEQUENCE的值，但是在第一次NEXTVAL初始化之后才能使用CURRVAL，否则会出错。

` 一次NEXTVAL会增加一次 SEQUENCE的值，所以如果你在同一个语句里面使用多个NEXTVAL，其值就是不一样的。`

 如果指定CACHE值，ORACLE就可以预先在内存里面放置一些sequence，这样存取的快些。cache里面的取完后，oracle自动再取一组 到cache。 使用cache或许会跳号， 比如数据库突然不正常down掉（shutdown abort),cache中的sequence就会丢失. 所以可以在create sequence的时候用nocache防止这种情况。

### Alter Sequence

```
拥有ALTER ANY SEQUENCE 权限才能改动sequence. 可以alter除start至以外的所有sequence参数.如果想要改变start值，必须 drop sequence 再 re-create。
```

例：
alter sequence SEQTEST maxvalue 9999999;

```
另： SEQUENCE_CACHE_ENTRIES参数，设置能同时被cache的sequence数目。
```

### Drop Sequence

```plsql
DROP SEQUENCE seqTest;
```

### Reset Sequence

有时候需要重置序列，所以要曲线的调用存储过程**重置序列**。

```plsql
create or Replace Procedure seq_Reset(v_seqname in  varchar2) as  n number(10);
                                                                                                      tsql varchar2(100);
Begin
  tsql := 'Select ' || v_seqname || '.nextval From dual';
  Execute Immediate tsql into n;
  --如果序列本身是初始状态则不进行数值计算
  if n <> 1 then
    n := -(n-1);
  end if;
  tsql := 'Alter Sequence ' || v_seqname || ' Increment By ' || n;
  Execute Immediate tsql;
  tsql := 'Select ' || v_seqname || '.nextval From dual';
  Execute Immediate tsql into n;
  tsql := 'Alter Sequence ' || v_seqname || ' Increment By 1';
  Execute Immediate tsql;
  End seq_Reset;
```

### Sample

```plsql
create sequence SEQ_ID
minvalue 1
maxvalue 99999999
start with 1
increment by 1
nocache
order;
```

建解发器代码为：

```plsql
create or replace trigger tri_test_id
  before insert on S_Depart   --S_Depart 是表名
  for each row
declare
  nextid number;
begin
  IF :new.DepartId IS NULLor :new.DepartId=0 THEN --DepartId是列名
    select SEQ_ID.nextval --SEQ_ID正是刚才创建的
    into nextid
    from sys.dual;
    :new.DepartId:=nextid;
  end if;
end tri_test_id;
```

 上面的代码就可以实现自动递增的功能了。

```plsql
注    :new 代表 数据改变后的新值，相对应的有 :old 原值
      := 代表 赋值
      :nextid表示引用sqlplus中定义的变量 
```

以上内容是在转载和总结的基础上写出来的，仅供参考。

 转载网站： [ORACLE SEQUENCE用法](http://www.cnblogs.com/hyzhou/archive/2012/04/12/2444158.html)
 参考文章： [Oracle创建自增字段方法-ORACLE SEQUENCE的简单介绍](http://blog.csdn.net/zhoufoxcn/article/details/1762351)

