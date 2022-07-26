---
title: 记一次MySQL回滚操作
date: 2022-05-16 10:54:57
tags:
---

### 问题描述
组内小伙伴在一次数据库变更中出现了一次误操作，批量修改了业务表某个必填字段导致产生了一堆脏数据，使得该业务模块不可用。而该模块被其他多个服务所依赖，造成了大规模服务不可用。所以必须进行sql回滚操作。

#### 解决方法
1. 获取binlog日志
回滚数据库方法的前提是MySQL开启了binlog日志，如果没有开启binlog就无法回滚。所以建议业务库都开启binlog。
开启binlog的方法
```
[mysqld]
server_id=1
log_bin=mysql-bin
binlog_format=ROW
expire_logs_days=7
```

登陆数据库后，执行
```
show master status;
+------------------+-----------+--------------+------------------+-------------------+
| File             | Position  | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+-----------+--------------+------------------+-------------------+
| mysql_bin.000081 | 356521886 |              |                  |                   |
+------------------+-----------+--------------+------------------+-------------------+
```
得知目前binlog记录的日志文件是mysql_bin.000081。
根据操作发生的时间，可以执行
```
mysqlbinlog -vv mysql_bin.000081 --start-datetime="2022-05-13 16:32:00" --stop-datetime="2022-05-13 16:32:46" --base64-output=decode-rows –v -d api > sql.log
```
如果不清楚业务发生时间，可以不加--start-datetime和--stop-datetime参数。-d 后的参数是指具体的database。执行命令后会得到sql.log文件

2. 解析binlog日志
打开后会看到里面的内容基本如下。
```
BEGIN
/*!*/;
# at 76029927
#220513 16:34:07 server id 208  end_log_pos 76030006 CRC32 0x8ab2f333 	Table_map: `center`.`order` mapped to number 126
# at 76030006
#220513 16:34:07 server id 208  end_log_pos 76038122 CRC32 0xfb726918 	Update_rows: table id 126
# at 76038122
#220513 16:34:07 server id 208  end_log_pos 76046334 CRC32 0x14478af4 	Update_rows: table id 126
# at 76046334
#220513 16:34:07 server id 208  end_log_pos 76046594 CRC32 0x1409de9a 	Update_rows: table id 126 flags: STMT_END_F
### UPDATE `center`.`order`
### WHERE
###   @1=3459 /* INT meta=0 nullable=0 is_null=0 */
###   @2=1 /* INT meta=0 nullable=0 is_null=0 */
###   @3=1 /* INT meta=0 nullable=0 is_null=0 */
###   @4='1644987397' /* VARSTRING(384) meta=384 nullable=0 is_null=0 */
###   @5=3121 /* INT meta=0 nullable=1 is_null=0 */
###   @6=2 /* TINYINT meta=0 nullable=0 is_null=0 */
###   @7=0 /* TINYINT meta=0 nullable=0 is_null=0 */
###   @8='2022-01-20 06:46:49' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @9='2022-01-20 06:46:49' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @10=1 /* INT meta=0 nullable=0 is_null=0 */
###   @11=1 /* INT meta=0 nullable=0 is_null=0 */
###   @12=0 /* TINYINT meta=0 nullable=0 is_null=0 */
### SET
###   @1=3459 /* INT meta=0 nullable=0 is_null=0 */
###   @2=1 /* INT meta=0 nullable=0 is_null=0 */
###   @3=1 /* INT meta=0 nullable=0 is_null=0 */
###   @4='1644987397' /* VARSTRING(384) meta=384 nullable=0 is_null=0 */
###   @5=3121 /* INT meta=0 nullable=1 is_null=0 */
###   @6=2 /* TINYINT meta=0 nullable=0 is_null=0 */
###   @7=0 /* TINYINT meta=0 nullable=0 is_null=0 */
###   @8='2022-01-20 06:46:49' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @9='2022-01-20 06:46:49' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @10=1 /* INT meta=0 nullable=0 is_null=0 */
###   @11=1 /* INT meta=0 nullable=0 is_null=0 */
###   @12=1 /* TINYINT meta=0 nullable=0 is_null=0 */
# at 76046594
#220513 16:34:07 server id 208  end_log_pos 76046625 CRC32 0xbf2d840e 	Xid = 885802622
COMMIT/*!*/;
# at 76046625
```
这一段便是一个sql执行的binlog记录。BEGIN代表事务开启，at 76029927代表此段sql在binlog中的偏移量，这是我们回滚sql的重要依据。从UPDATE开始，便是误操作的sql，可以看到我们对center库中的order表进行了一次update操作。COMMIT代表了事务已被提交，sql便在数据库中执行了。COMMIT后的at 76046625代表事务结束时的偏移量。
至此，我们已经得到了sql执行前和执行后的position：76029927&76046625。

3. 获得回滚SQL
安装开源工具binlog2sql。binlog2sql是一款简单易用的binlog解析工具，其中一个功能就是生成回滚SQL。
```bash
git clone https://github.com/danfengcao/binlog2sql.git 
&& cd binlog2sql
```
然后执行
```
python binlog2sql.py -h127.0.0.1 -P3306 -uadmin -p'admin' -dcenter -torder --start-file='mysql-bin.000081' --start-position=76029927 --stop-position=76046625 -B > rollback.sql
```
在工程根目录下即得到一个rollback.sql文件
与业务方确认回滚sql没问题，执行回滚语句。登录mysql，确认回滚成功。