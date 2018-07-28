---
title: Mysql（剔除冗余数据）
tags:
  - Mysql
categories: Mysql
---

## Mysql 剔除冗余数据
>事出有因，解释起来太麻烦了，举个例子，简化说下问题。

| user\_id| passport\_id |
| ------------- |:-------------:|
| 1 | 1 |
| 2 | 2 |
| 4 | 2 |
| 3 | 3 |
| 5 | 3 |
| 6 | 3 |
<!-- more -->

我有一张user表，但是一不小心被混进去了脏数据，目前的情况是：一个passport_id可能对应多个user_id。现在我的目标是清理脏数据。已知条件：<br>
1.  每个passport_id仅对应一个user_id；
2.  有效的user_id是最小的

根据条件，上述(user_id, passport_id)中(4, 2), (5, 3), (6, 3)是多余的。


## 解决
### 解决思路
很明显，如果passport_id对应一个user_id，则为有效数据；如果passport_id对应多条数据，则user_id最小的有效，其它无效。所以需要删除的是passport_id对应多条数据，并且user_id不是最小的。

### 确定基准
```
select passport_id,count(*) as total, min(user_id) as min_user_id from `tk_user`  group by passport_id having total > 1
```
![基准](https://img.ryoma.top/MysqlSecond/1.png)

查找出需要清理数据的基础数据，如上，passport_id=2时，user_id>2的都需要清除。

### 查找出需要删除的数据，并建临时表存储
```
create table 2017_08_11_tmp_user as (select b.user_id from
(select passport_id,count(*) as total,min(user_id) as min_user_id from `tk_user`  group by passport_id having total>1) as a,`tk_user` b
where a.passport_id=b.passport_id and a.min_user_id<b.user_id);
```

![临时表](https://img.ryoma.top/MysqlSecond/2.png)

### 基于临时表删除数据
```
DROP TABLE IF EXISTS `2017_08_11_tmp_user`;
create table 2017_08_11_tmp_user as (select b.user_id from
(select passport_id,count(*) as total,min(user_id) as min_user_id from `tk_user`  group by passport_id having total>1) as a,`tk_user` b
where a.passport_id=b.passport_id and a.min_user_id<b.user_id);

delete from `tk_user` where tk_user.user_id in (select user_id from 2017_08_11_tmp_user); 
drop table 2017_08_11_tmp_user;
```
![结果](https://img.ryoma.top/MysqlSecond/3.png)

## 结语
工作时突然遇到这个问题，代码的发布需要走的流程太多，故而选择了直接执行sql。当时，写代码的话，方法是更简单的。不过，sql还是超级强大的。

