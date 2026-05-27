# 幂等插入语句
当插入数据出现主键和唯一索引冲突是，会改为更新操作，否则就是插入操作。
```sql
INSERT INTO t_log (user_name, log_date, content) VALUES ('Tom', '2023-10-01', '...')
    ON DUPLICATE KEY UPDATE content = VALUES(content);
```
