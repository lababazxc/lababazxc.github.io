---
title: python调用shell实践
date: 2022-04-11 11:16:23
tags:
---

#### 问题描述
自动化测试中，存在一些测试用例数据，这些数据被存储在MySQL当中。在新的环境运行自动化测试之前，需要将这些用例数据统一写入MySQL当中。
现在已有的文件：sql存储脚本，如testcase_1.sql；调用sql的shell脚本。
因为pymysql调用执行sql文件不方便的原因，选择用python调用shell的方式来刷库。

---

#### 解决方案
1. os.system()
    