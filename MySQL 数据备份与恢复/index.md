## MySQL 数据备份与恢复 
### 备份的类型

数据备份会分为两大类型：`物理 (原始) 备份` (physical (raw) backups) 和`逻辑备份` (logical backups) ，你可以在[这里](https://dev.mysql.com/doc/mysql-backup-excerpt/8.0/en/backup-types.html)找到更多信息。

* `物理备份`由存储数据库内容的目录和文件的原始副本组成。这种类型的备份适用于需要在出现问题时，需要快速恢复的大型、重要的数据库。
* `逻辑备份`保存以逻辑数据库结构 (CREATE DATABASE、 CREATE TABLE 语句) 和内容 (INSERT 语句或分隔文本文件) 表示的信息。这种类型的备份适用于较小的数据量，您可以在其中编辑数据值或表结构，或者在不同的机器架构上重新创建数据。

简单来讲，物理备份就是拷贝数据库原始文件，而逻辑备份通过工具 (mysqldump) 来把数据库内容通过 `sql` 语句导出来。

#### 在线 vs 离线

根据导出的时，是否需要关闭 MySQL 服务，又出分`在线备份` (online backups) 和`离线备份` (offline backups) ，这两种备份方式也被称为`热备份` (hot backups) 和`冷备份` (cold backups) 。

* 离线备份
  * 由于备份期间服务器不可用，客户端可能会受到不利影响。因此，此类备份通常取自可以脱机而不会损害副本可用性；
  * 备份过程更简单，因为客户端活动不可能干扰。
* 在线备份
  * 备份对其他客户端的干扰较小，其他客户端可以在备份期间连接到 MySQL 服务器，并且可能能够根据他们需要执行的操作访问数据；
  * 必须注意施加适当的锁定，以便不会发生会损害备份完整性的数据修改。MySQL Enterprise Backup 产品会自动执行此类锁定。

#### 本地 vs 远程

根据备份是否在同一台主机上执行，可以分为`本地备份` (local backup) 和`远程备份` (remote backup) 。

* 通常物理备份在主机上本地执行，同时方便执行主机离线操作，即便拷贝数据库原始文件的目的地在远端主机；
* mysqldump 工具可以连接到本地或远程服务器，完成本地或远程转储。

#### 快照备份

`快照备份` (snapshot backups) 指一些文件系统提供拍摄“快照”。它们在给定的时间点提供文件系统的逻辑副本，而不需要整个文件系统的物理副本。例如，实现可能使用写时复制技术，以便只需要复制在快照时间之后修改的文件系统的部分。

MySQL 本身不提供获取文件系统快照的能力。它可通过 Veritas、LVM 或 ZFS 等第三方解决方案获得。

#### 全量备份 vs 增量备份

`全量备份` (full backup) 包括在给定`时间点`由 MySQL 服务器管理的所有数据。MySQL 有不同的方法来执行全量备份，例如本节前面描述的那些。

`增量备份` (incremental backup) 包括在给定`时间跨度`内 (从一个时间点到另一个时间点) 对数据所做的更改。通过启用服务器的`二进制日志` (binary log) 来实现增量备份，服务器使用它来记录数据更改。

### 备份前期工作

下面是为`增量备份`的准备工作，MySQL 二进制日志对于恢复很重要，因为它们构成了一组增量备份。如果你不准备采用`增量备份`，那么可以略过本小节。

备份前，我们需要确认 MySQL 是否已经开启了二进制日志功能。若未开启，下面几个步骤将介绍如何开启它。

**1.查看包含 `bin` 的配置变量状况。**

你可以通过 `mysqld --verbose --help`，查看所有配置项，在登录 MySQL 后，也可以通过下面的命令获取想要的配置项。

``` sh
mysql> show variables like '%bin%';
+--------------------------------------------+----------------------+
| Variable_name                              | Value                |
+--------------------------------------------+----------------------+
| bind_address                               | *                    |
| binlog_cache_size                          | 32768                |
| binlog_checksum                            | CRC32                |
| binlog_direct_non_transactional_updates    | OFF                  |
| binlog_error_action                        | ABORT_SERVER         |
| binlog_format                              | ROW                  |
| binlog_group_commit_sync_delay             | 0                    |
| binlog_group_commit_sync_no_delay_count    | 0                    |
| binlog_gtid_simple_recovery                | ON                   |
| binlog_max_flush_queue_time                | 0                    |
| binlog_order_commits                       | ON                   |
| binlog_row_image                           | FULL                 |
| binlog_rows_query_log_events               | OFF                  |
| binlog_stmt_cache_size                     | 32768                |
| binlog_transaction_dependency_history_size | 25000                |
| binlog_transaction_dependency_tracking     | COMMIT_ORDER         |
| innodb_api_enable_binlog                   | OFF                  |
| innodb_locks_unsafe_for_binlog             | OFF                  |
| log_bin                                    | OFF                  |
| log_bin_basename                           |                      |
| log_bin_index                              |                      |
| log_bin_trust_function_creators            | OFF                  |
| log_bin_use_v1_row_events                  | OFF                  |
| log_statements_unsafe_for_binlog           | ON                   |
| max_binlog_cache_size                      | 18446744073709547520 |
| max_binlog_size                            | 1073741824           |
| max_binlog_stmt_cache_size                 | 18446744073709547520 |
| sql_log_bin                                | ON                   |
| sync_binlog                                | 1                    |
+--------------------------------------------+----------------------+
29 rows in set (0.01 sec)
```

可以看到 `log_bin` 的配置为 `OFF`，说明 MySQL 并未启动二进制日志。

如果你执行了 `show binary logs;` 的命令，会得到下面结果：

```sh
mysql> show binary logs;
ERROR 1381 (HY000): You are not using binary logging
```

**2.查看二进制目录的存储位置**

```sh
mysql> show variables like 'datadir';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.00 sec)

```

**3.启动二进制日志**

通常把自定义的用户配置放在 `conf.d` 目录下，并命名为 `your_config_name.cnf` 格式。

我们在这个文件里加入下面几行配置：

``` text
[mysqld]
log_bin                = /var/log/mysql/mysql-bin.log
server-id              = 1
expire_logs_days       = 10
max_binlog_size        = 100M
binlog_format          = mixed
```

**4.再次查看二进制日志**

``` sh
mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       154 |
+------------------+-----------+
1 row in set (0.00 sec)

```

**5.访问二进制日志**

``` sh
> mysqlbinlog /var/log/mysql/mysql-bin.000001

/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#220104  9:03:29 server id 1  end_log_pos 123 CRC32 0x593c2727  Start: binlog v 4, server v 5.7.35-log created 220104  9:03:29 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
YQ3UYQ8BAAAAdwAAAHsAAAABAAQANS43LjM1LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAABhDdRhEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
AScnPFk=
'/*!*/;
# at 123
#220104  9:03:29 server id 1  end_log_pos 154 CRC32 0x7499a68e  Previous-GTIDs
# [empty]
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

### 锁

在执行备份时，通常需要进行`锁表`操作。

进行备份时，我们不希望还有客户端对数据库进行`写操作`，则需要获取表的`读锁`，此时所有会话都只能进行`读`访问。在进行恢复时，需要获取表的`写锁`，此时只有持有锁的会话可以读写表，其它会话都不能访问。 (本段只提到表锁，你可以在[这里](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)获取更多锁相关信息)

通过下面的命令，关闭所有打开的表并使用`全局读锁`锁定`所有数据库`的`所有表`。在下一节内容，使用 **mysqldump** 备份时，内部首先也会执行该命令。

``` sql
FLUSH TABLES WITH READ LOCK
```

### 备份策略

这里我们将介绍`逻辑备份`，并采用`全量备份 + 增量备份`的备份策略。

#### 全量备份

全量备份都将使用 [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html) 工具，该工具由是官方提供的并可以 CSV 格式输出以及其他分隔符的文本或 XML 格式。

如果我们要对某个数据库进行全量备份，使用以下语法：

```sh
mysqldump [options] db_name [tbl_name ...]
mysqldump [options] --databases db_name ...
mysqldump [options] --all-databases
```

 `mysqldump` 允许一张[组]表或者一个[组]数据库以及整个 MySQL 服务器进行备份，也可以叫做`转储`。

#### 例子

假如有一个数据库叫 `publish_system_test`，我们对其进行全量备份，那么可以写如下命令：

``` sh
mysqldump -u root -p -h localhost --flush-logs --master-data=2 --single-transaction --databases publish_system_test > "publish_system_test_backup_$(date +"%Y%m%d_%H%M%S").sql"
```

在导出的文件名称命名上，我们使用 `date` 命令在文件尾部添加备份日期，这方便在恢复时能快速识别备份日期。`.sql`文件也被叫做`转储文件` (dump file) 。

上面的执行，会对所有该数据库的表加上`读锁`，然后导出 SQL 语句，这意味着在备份期间客户端对该数据库只有读取访问能力。

我们可以看到以下部分内容：

``` mysql
LOCK TABLES `project` WRITE;
/*!40000 ALTER TABLE `project` DISABLE KEYS */;
INSERT INTO `project` VALUES (1,'hello','good','我是项目描述',NULL,'weapp,web',685,'卢敏','http://xxx.xxx.com.cn/lumin/mypa222th','porjecsasaa',NULL,'2022-01-04 07:37:33.735181','卢敏',1221,NULL,NULL,NULL,NULL);
/*!40000 ALTER TABLE `project` ENABLE KEYS */;
UNLOCK TABLES;
```

从 SQL 语句可以看到，当执行该脚本时，在执行插入 (INSERT) 语句之前会先进行`锁写表`，完成插入任务之后再进行`解锁`。

[--master-data](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html#option_mysqldump_master-data)参数，该参数将在导出文件里记录转储服务器的二进制日志坐标 (文件名和位置) ，这个选项要求 RELOAD 权限，并且二进制日志功能必须是启用状态。

你可以看到下面的因为该参数导出的语句：

``` sql
--
-- Position to start replication or point-in-time recovery from
--

-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=4; 
```

注释代码已经解释了它的用处，用于二进制日志`副本复制`或[时间点恢复](https://dev.mysql.com/doc/mysql-backup-excerpt/8.0/en/point-in-time-recovery-binlog.html)(point-in-time)

`--master-data`默认参数值为 1，当指定为 2 时，会以注释状态输出。

[--single-transaction](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html#option_mysqldump_single-transaction)参数，该参数会把`事务隔离模式` (transaction isolation mode) 设置为`可重复读取` (REPEATABLE READ) ，并且在转储之前发送 `START TRANSACTION` SQL 语句给服务器，表示即将启动一个事务。

通过这个参数，能够保证转储时上数据库的`一致性`。该参数只对事务性表起作用，例如：InnoDB 表，而像 MyISAM 或者 MEMORY 表转储时状态可能发生改变。

当有一个包含`--single-transaction`选项的转储进行中，为了确保输出一个有效的转储文件 (正确的表内容和二进制日志坐标) ，不应该有其它数据库连接执行以下语句：`ALTER TABLE`, `CREATE TABLE` , `DROP TABLE` , `RENAME TABLE` , `TRUNCATE TABLE`，一致读不会隔离这些语句。

[--flush-logs](https://dev.mysql.com/doc/refman/8.0/en/flush.html#flush-logs)参数，该参数会使数据目录包含一个新的二进制日志文件。也就是说日志文件 `mysql-bin.000002` 是新创建的，后面的对数据库更改的日志都将从该文件开始写入。

#### 增量备份

要进行增量备份，我们需要保存增量更改。在 MySQL 中，这些更改在二进制日志中表示，因此 MySQL 服务器应始终使用`--log-bin`选项启动。启用二进制日志记录后，服务器的每个数据更改都会写入日志文件。

增量备份的本质是创建一个新的日志文件，后续的更改都将从新的日志文件开始写入。所以我们可以定期执行刷新日志操作来创建增量备份。

通过下面的 `mysqladmin` 命令执行刷新日志操作：

``` sh
mysqladmin flush-logs
```

或者使用 [FLUSH](https://dev.mysql.com/doc/refman/8.0/en/flush.html#flush-logs) SQL语句：

``` sql
flush logs;
```

并且你可以通过以下命令获取当前记录的日志状态：

``` sql
show master status;
```

### 恢复备份

恢复备份首先要做的是恢复最后一个全量备份，再根据最后一个全量备份的时间点找到`大于`该时间点并`小于`故障发生之前的时间点之间的所有增量备份。

完全备份的恢复方式是把转储文件通过 `mysql` 命令载入，并且你还可以指定一个 host 使数据载入到另一台远程 SQL 服务器。

``` sh
mysql --host=host_name -u root -p < publish_system_test_backup_20220105_030044.sql
```

而后，我们通过找到的二进制日志文件，通过 [mysqlbinlog](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html) 工具进行恢复：

``` sh
mysqlbinlog mysql-bin.000001 mysql-bin.000001 | mysql
```

参考资料：

\> [https://dev.mysql.com/doc/mysql-backup-excerpt/8.0/en/backup-types.html](https://dev.mysql.com/doc/mysql-backup-excerpt/8.0/en/backup-types.html)

\> [https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html#lock-tables-and-triggers](https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html#lock-tables-and-triggers)
