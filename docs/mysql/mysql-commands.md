# MySQL

## 命令行 cli 连接 MySQL Server

```bash
# 在命令行里写明文密码是不安全的，例如 mysql -h$Host -u$Username -p$Password 会报 [Warning] Using a password on the command line interface can be insecure.
# 
# 安全的方法（选其一即可）：
# * 使用 mysql_config_editor 将身份凭证存储在 .mylogin.cnf 加密登录路径文件中，参考 https://dev.mysql.com/doc/refman/8.0/en/mysql-config-editor.html
# * 在命令行里使用 -p 或 --password 选项，让客户端程序提示输入密码
# * 在 ~/.my.cnf 文件中配置明文密码，详细参考 https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html#:~:text=%5Bclient%5D%0Apassword%3Dpassword
# 
MYSQL_PWD=$Password mysql -h$Host -u$Username  # 正确的方式是将 MYSQL_PWD 环境变量在其他地方进行设置
```

## 命令行 cli 执行 MySQL 命令（不进入交互模式）

```bash
mysql -e "SELECT VERSION();"
```

## 查看某库中某表的自增 ID 计数器

```sql
SELECT `AUTO_INCREMENT` 
FROM  INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA = 'YOUR_DATABASE' AND TABLE_NAME = 'YOUR_TABLE';
-- WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'YOUR_TABLE';
```

## 根据旧表建新表

```sql
CREATE TABLE new_table
LIKE old_table;            -- LIKE 会包含列信息和索引信息

-- CREATE TABLE new_table
-- SELECT *                -- SELECT 只会复制列信息，不会复制索引信息
-- FROM old_table;      
```

## 查看 server id

```sql
SELECT @@server_id;
```

## 查看当前的 connections 连接数

```bash
mysql -e "SHOW PROCESSLIST;" | tail -n +2 | wc -l
```

## 最大连接数

```sql
SELECT @@GLOBAL.max_connections;       -- 服务器上同时存在的最大连接数
SELECT @@GLOBAL.max_user_connections;  -- 单个用户可以同时建立的连接数
```

## 慢查询日志

```sql
SELECT @@GLOBAL.slow_query_log;                 -- 启用或禁用慢查询日志功能（有效值：ON/1、OFF/0）
SELECT @@GLOBAL.slow_query_log_file;            -- 慢查询日志文件存储位置
SELECT @@GLOBAL.long_query_time;                -- 设置慢查询的阈值时间（单位：秒）
SELECT @@GLOBAL.log_queries_not_using_indexes;  -- 是否记录那些未使用索引的查询语句到慢查询日志中（有效值：ON/1、OFF/0）
```

若开启了慢查询日志，需要注意慢查询日志文件的定期备份与清理，避免浪费磁盘空间

```bash
cat /dev/null > $SLOW_QUERY_LOG_FILE
```

## 查看 innodb_buffer_pool_size 内存大小

```sql
SELECT  @@innodb_buffer_pool_size/1024/1024/1024;  -- 以 G 为单位
```

## 查看实例当前拥有的 binlog

```sql
SHOW BINARY LOGS;
```

## 在线查看 binlog 内容

```sql
SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,], row_count];
```

## 时间类型与 INT、VARCHAR 等类型相互转换

```sql
-- NOW() -> VARCHAR
SELECT DATE_FORMAT(NOW(6), '%Y-%m-%d %H:%m:%s.%f');

-- CURRENT_TIMESTAMP -> VARCHAR
SELECT DATE_FORMAT(CURRENT_TIMESTAMP, '%Y-%m-%d %H:%m:%s.%f');

-- VARCHAR -> DATETIME
--   %W: Weekday name from Sunday to Saturday.
--   %d: Day of the month as a numeric value from 01 to 31.
--   %m: Month as a numeric value from 01 to 12.
--   %Y: Year as a 4-digit numeric value (yyyy).
--   %T: Time in the 24-hour format (hh:mm:ss).
SELECT STR_TO_DATE('2023-07-13 03:49:11', '%Y-%m-%d %T');

-- BIGINT -> DATETIME
SELECT FROM_UNIXTIME(1680873114);           -- 毫秒
SELECT FROM_UNIXTIME(1680873114000 / 1000); -- 微秒
```