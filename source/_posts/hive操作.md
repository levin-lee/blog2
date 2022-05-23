---
title: hive操作
date: 2020-10-11 21:02:40
tags:
- hive
- 编程
description: Hive的基本用法
cover: https://pic3.zhimg.com/80/v2-049af85e7a7f0d35d2ceba99b283c8a2_1440w.jpg
---
**创建数据库例子**
```hive
hive> create database if not exists user_db;
```

**查看数据库定义**
```hive
hive> describe database user_db;
```

**查看数据库列表**
```hive
hive> show databases; 
```

**删除数据库**
```hive
hive> drop database if exists testdb cascade;
```

**切换当前数据库**
```hive
hive> use user_db;
```

**创建普通表**
1)第一种
```hive
hive> create table if not exists userinfo  
    (
    userid int,
    username string,
    cityid int,
    createtime date    
    )
    row format delimited fields terminated by '\t'
    stored as textfile;
```
row format delimited fields terminated by '\t' 是指定列之间的分隔符；stored as textfile是指定文件存储格式为textfile。

2）第二种
create table as select 方式：根据查询的结果自动创建表，并将查询结果数据插入新建的表中。
3）第三种
create table like tablename1 方式：是克隆表，只复制tablename1表的结构。复制表和克隆表会在下面的Hive数据管理部分详细讲解。

**创建外部表**
外部表是没有被hive完全控制的表，当表删除后，数据不会被删除。
```hive
hive> create external table iislog_ext (
    ip string,
    logtime string    
    );
```

**创建分区表**
```hive
create table user_action_log
(
companyId INT comment   '公司ID',
userid INT comment   '销售ID',
originalstring STRING comment   'url', 
)
partitioned by (dt string)
row format delimited fields terminated by ','
stored as textfile;
```

**创建桶表**
```hive
create table user_leads
(
leads_id string,
user_id string,
user_id string,
user_phone string,
user_name string,
create_time string
)
clustered by (user_id) sorted by(leads_id) into 10 buckets 
row format delimited fields terminated by '\t' 
stored as textfile;
```

**查看简单定义**
```hive
describe userinfo;
```

**查看表详细信息**
```hive
describe formatted userinfo;
```

**修改表名**
```hive
alter table userinfo rename to user_info;
```

**添加字段**
```hive
alter table user_info add columns (provinceid int );
```

**修改字段**
```hive
alter table user_info replace columns (userid int,username string,cityid int,joindate date,provinceid int);
```

将本地文本文件内容批量加载到Hive表中：
——要求文本文件中的格式和Hive表的定义一致
```hive
load data local inpath '/home/hadoop/userinfodata.txt' overwrite into table user_info;
```

**加载到分区表**
```hive
load data local inpath '/home/hadoop/actionlog.txt' overwrite into table user_action_log PARTITION (dt='2017-05-26’);
```

**导入分桶表**
```hive
set hive.enforce.bucketing = true;
insert overwrite table user_leads select * from  user_leads_tmp;
```

**导出数据**
```hive
insert overwrite local directory '/home/hadoop/user_info.bak2016-08-22 '
select * from user_info;
```

**插入数据的表是分区表**
```hive
insert overwrite table user_leads PARTITION (dt='2017-05-26') 
select * from  user_leads_tmp;
```

**复制表**
```hive
create table user_leads_bak
row format delimited fields terminated by '\t'
stored as textfile
as
select leads_id,user_id,'2016-08-22' as bakdate
from user_leads
where create_time<'2016-08-22’;
```

**克隆表**
——克隆表时会克隆源表的所有元数据信息，但是不会复制源表的数据
```hive
create table user_leads_like like  user_leads;
```

|操作符|说明|操作符|说明|
|:----|:----|:----|:----|
|A=B|A等于B就返回true，适用于各种基本类型|A<=>B|都为Null则返回True，其他和=一样 |
|A<>B|不等于|A!=B|不等于|
|A<=B|小于等于|A>B|大于|
|A>=B|大于等于|A Between B And C|筛选A的值处于B和C之间|
|A Not Between B And C|筛选A的值不处于B和C之间|A Is NULL|筛选A是NULL的|
|A Is Not NULL|筛选A值不是NULL的|A Link B | %一个或者多个字字符|
|A Not Like B |%一个或者多个字符_一个字符 |A RLike B |正则匹配 |

**排序**
Sort By——Hive中尽量不要用Order By，除非非常确定结果集很小
```hive
select * from user_leads sort by user_id
```

## 日期函数
1、UNIX时间戳转日期函数：from_unixtime
转化UNIX时间戳（从1970-01-01 00:00:00 UTC到指定时间的秒数）到当前时区的时间格式
```hive
hive> select from_unixtime(1323308943,'yyyyMMdd') fromlxw_dual;
```

2、日期时间转日期函数：to_date
```hive
hive> select to_date('2011-12-08 10:03:01') from lxw_dual;
```

3、日期转年\月\日函数：year——month——day
```hive
hive> select year('2011-12-08 10:03:01') from lxw_dual;
```

4、日期比较函数：datediff
```hive
hive> select datediff('2012-12-08','2012-05-09') from lxw_dual;
```

5、日期增加、减少函数：date_add/date_sub
```hive
hive> select date_add('2012-12-08',10) from lxw_dual;
```

## 字符串函数

1、字符串反转函数：reverse
```hive
hive> select reverse(abcedfg’) from lxw_dual;
```

2、字符串连接函数：concat
```hive
hive> select concat(‘abc’,'def’,'gh’) from lxw_dual;
```

3、带分隔符连接函数：concat_ws
```hive
hive> select concat_ws(',','abc','def','gh') from lxw_dual;
```

4、字符串截取函数：substr、substring
返回字符串A从start位置到结尾的字符串
```hive
hive> select substr('abcde',3) from lxw_dual;
```


5、字符串转大写和小写：upper、ucase / lower、lcase
```hive
select lcase('abSEd') from lxw_dual;
```

6、去空格函数
```hive
hive> select trim(' abc ') from lxw_dual;
```

7、正则表达式替代函数：regexp_replcae
```hive
hive> select regexp_replace('foobar', 'oo|ar', '') from lxw_dual;
```

8、json解析函数：get_json_object
```hive
get_json_object(string json_string, string path)
```
9、分隔字符串函数：split 返回数组
```hive
hive> select split('abtcdtef','t') from lxw_dual;
```

