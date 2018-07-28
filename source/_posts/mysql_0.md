---
title: Mysql（清除空数据）
date: 2017/12/31 12:00:00
tags:
  - Mysql
  - replace
categories: Mysql
---

## Mysql 清除空格 换行
- 最近从后台拉数据的时候，突然发现数据中有很多格式很奇怪的数据。比如拉出来的数据保存至excel中的时候，会有一些奇怪的换行。原因当然是存储的时候没有对数据进行严格的处理，这部分由于使用的是公司的C++框架，之前使用的时候也忽略了这类数据的处理。（之前在使用PHP的时候，数据会先统一处理，保证在action中的数据都是可以直接使用的）

<!-- more -->

### replace
- 使用replace替换 空格 & 换行
```
SELECT REPLACE(REPLACE(A.company_name, CHAR(10), ''), CHAR(13),''), A.quqi_id, B.phone  from sys_company A LEFT JOIN
sys_passport B
on A.creator_passport_id = B.passport_id;
```

## 说明
类似之前写的[windows & linux小功能](/2017/11/12/windows%20&%20linux小功能/)，之后的mysql小技巧也会放在这个文件。有些东西虽然很简单，在网上也很方便能查到，但是自己写出来，至少会加深印象~~~