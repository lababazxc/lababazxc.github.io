---
title: MySQL数据类型简析
date: 2022-08-13 09:26:46
tags: 数据库集合
---

MySQL是现在最常用的关系型数据库之一，本文对MySQL可用的所有字段类型做一个简单的总结。

### 数据类型概览

| 数据类型      | 描述 | 示例 |
| :--:   |    :------:   |          :-----: |
|BOOLEAN|  布尔值|1|
|BIT|位值(1～64)|b01|
|TINYINT|整型(-128～127)|10|
|TINYINT UNSIGNED|整型(0～255)|255|
|SMALLINT|整型(-32768～32767)|123|
|SMALLINT UNSIGNED|整型(0～65535)|65535|
|MEDIUMINT|整型(-8388608～8388607)|8388607|
|MEDIUMINT UNSIGNED|整型(0～16777215)|16777215|
|INT|整型(-2147483648～2147483647)|2147483647|
|INT UNSIGNED|整型(0～4294967295)|4294967295|
|INTEGER		|int的装箱类型|-|
|BIGINT|整型|-1235|
|BIGINT UNSIGNED|整型|12345|
|REAL|浮点型|12.23|
|FLOAT|浮点数|12.23|
|FLOAT UNSIGNED|无符号浮点|12.23|
|DECIMAL|定点数|12.23|
|DECIMAL UNSIGNED|无符号定点数|12.23|
|NUMERIC|decimal的一种实现，使用语法同decimal|12.23|
|DOUBLE|浮点数|12.23|
|DOUBLE UNSIGNED|浮点|12.23|
|STRING|字符串总称|-|
|VARCHAR|长字符串|这是一个字符串|
|CHAR|字符串|这是一个字符串|
|TIMESTAMP|contain both date and time parts.|1970-01-01 00:00:01 UTC|
|DATETIME|YYYY-MM-DD hh:mm:ss|1000-01-01 00:00:00|
|DATE|YYYY-MM-DD|1000-01-01|
|TIME|               hh:mm:ss               |       -838:59:59        |
|YEAR|YYYY|1901～2155|
|BLOB|二进制大对象，存储不定长度的数据||
|TINYBLOB|                 二进制大对象，存储不定长度的数据                 ||
|MEDIUMBLOB|二进制大对象，存储不定长度的数据||
|LONGBLOB|二进制大对象，存储不定长度的数据|-|
|TINYTEXT|二进制大对象，存储不定长度的数据|-|
|TEXT|非二进制字符串|-|
|MEDIUMTEXT|非二进制字符串|-|
|LONGTEXT|非二进制字符串|-|
|BINARY|以字节进行存储的字符串，不指定大小默认1|-|
|VARBINARY|以字符进行存储的字符串|-|
|JSON|字符串|'{"id": 1}'|
|ENUM|枚举|ENUM(1,2)|
|SET|字符串|                         |
|GEOMETRY||POINT(103 35)|

#### 数字型字段
数据型字段一共可分为位值（bit）、整型、定点型、浮点型。其中bool/boolean为tinyint(整型)的同义词。
