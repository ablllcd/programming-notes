# PostgreSQL 概念解释

## 登录
 
### Peer Authentication
PostgreSQL 默认使用 Peer Authentication（对本地连接进行用户身份验证），它要求：Linux 系统用户名必须与 PostgreSQL 数据库用户名一致。

为了避免每次连接都需要指定用户名，可以通过以下两种方式解决：
1. 使用 `sudo -i -u postgres` 切换到 `postgres` 用户，然后直接运行 `psql` 连接数据库。
2. 修改 `pg_hba.conf` 文件，将本地连接的认证方式改为 `md5`，然后重启 PostgreSQL 服务。这样可以使用任何数据库用户进行连接，但需要提供密码。

## 数据库

* 每个数据库必需有一个所有者（owner），通常是创建数据库的用户。

# PostgreSQL 基本操作指南

## 0 连接数据库

```
psql -U [username] -d [database]
```

## 1. 数据库基本操作

### 1.1 创建和删除数据库

```sql
-- 创建数据库
CREATE DATABASE database_name;

-- 删除数据库
DROP DATABASE database_name;

-- 列出所有数据库
\l

-- 连接到数据库
\c database_name
```

### 1.2 查看连接信息

```sql
-- 查看当前数据库
SELECT current_database();

-- 查看当前用户
SELECT current_user;

-- 查看详细连接信息
\conninfo
```

### 1.3 数据库的Owner操作

```sql
-- 查看数据库owner
SELECT datname, pg_catalog.pg_get_userbyid(datdba) AS owner FROM pg_database;

-- 修改数据库owner
ALTER DATABASE database_name OWNER TO new_owner;
```


## 2. 表操作

### 2.1 创建和删除表

```sql
-- 创建表
CREATE TABLE table_name (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    age INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 删除表
DROP TABLE table_name;

-- 查看所有表
\dt

-- 查看表结构
\d table_name
```

### 2.2 修改表结构

```sql
-- 添加列
ALTER TABLE table_name ADD COLUMN new_column_name data_type;

-- 删除列
ALTER TABLE table_name DROP COLUMN column_name;

-- 修改列类型
ALTER TABLE table_name ALTER COLUMN column_name TYPE new_data_type;
```

## 3. 数据操作（CRUD）

### 3.1 插入数据

```sql
-- 插入单条数据
INSERT INTO table_name (name, age) VALUES ('张三', 25);

-- 插入多条数据
INSERT INTO table_name (name, age) VALUES
    ('李四', 30),
    ('王五', 35);
```

### 3.2 查询数据

```sql
-- 查询所有数据
SELECT * FROM table_name;

-- 条件查询
SELECT name, age FROM table_name WHERE age > 20;

-- 排序
SELECT * FROM table_name ORDER BY age DESC;

-- 分页
SELECT * FROM table_name LIMIT 10 OFFSET 20;
```

### 3.3 更新数据

```sql
-- 更新数据
UPDATE table_name SET age = 26 WHERE name = '张三';

-- 批量更新
UPDATE table_name SET status = 'active' WHERE age > 20;
```

### 3.4 删除数据

```sql
-- 删除数据
DELETE FROM table_name WHERE name = '张三';

-- 清空表
TRUNCATE TABLE table_name;
```

## 4. 索引操作

### 4.1 创建和删除索引

```sql
-- 创建索引
CREATE INDEX index_name ON table_name (column_name);

-- 创建唯一索引
CREATE UNIQUE INDEX index_name ON table_name (column_name);

-- 删除索引
DROP INDEX index_name;
```

## 5. 约束操作

### 5.1 添加约束

```sql
-- 添加主键
ALTER TABLE table_name ADD PRIMARY KEY (id);

-- 添加外键
ALTER TABLE table_name
ADD CONSTRAINT fk_name
FOREIGN KEY (column_name)
REFERENCES other_table(column_name);

-- 添加唯一约束
ALTER TABLE table_name ADD UNIQUE (column_name);

-- 添加非空约束
ALTER TABLE table_name ALTER COLUMN column_name SET NOT NULL;
```

## 6. 视图操作

### 6.1 创建和删除视图

```sql
-- 创建视图
CREATE VIEW view_name AS
SELECT * FROM table_name WHERE condition;

-- 删除视图
DROP VIEW view_name;
```

## 7. 用户和权限管理

### 7.1 用户操作

```sql
-- 创建用户
CREATE USER username WITH PASSWORD 'password';

-- 删除用户
DROP USER username;

-- 修改密码
ALTER USER username WITH PASSWORD 'new_password';
```

### 7.2 权限管理

```sql
-- 授予权限
GRANT SELECT, INSERT ON table_name TO username;

-- 撤销权限
REVOKE SELECT ON table_name FROM username;
```

## 8. 备份和恢复

### 8.1 备份数据库

```bash
# 备份整个数据库
pg_dump database_name > backup.sql

# 备份特定表
pg_dump -t table_name database_name > table_backup.sql
```

### 8.2 恢复数据库

```bash
# 恢复数据库
psql database_name < backup.sql
```

```bash
# 加载tar文件到当前数据库
pg_restore -U username -d database_name /path/to/backup.tar
```

## 9. 事务操作

### 9.1 基本事务

```sql
-- 开始事务
BEGIN;

-- 提交事务
COMMIT;

-- 回滚事务
ROLLBACK;
```

## 10. 常用函数

### 10.1 字符串函数

```sql
-- 字符串长度
SELECT LENGTH('hello');

-- 大小写转换
SELECT UPPER('hello');
SELECT LOWER('HELLO');

-- 字符串连接
SELECT CONCAT('hello', ' ', 'world');
```

### 10.2 日期函数

```sql
-- 当前日期
SELECT CURRENT_DATE;

-- 当前时间戳
SELECT CURRENT_TIMESTAMP;

-- 提取日期部分
SELECT EXTRACT(YEAR FROM CURRENT_DATE);
```

### 10.3 数学函数

```sql
-- 四舍五入
SELECT ROUND(3.14159, 2);

-- 绝对值
SELECT ABS(-10);

-- 随机数
SELECT RANDOM();
```

## 11. 实用技巧

### 11.1 查看系统信息

```sql
-- 查看版本
SELECT version();

-- 查看表空间
\db

-- 查看所有模式
\dn
```

### 11.2 性能优化

```sql
-- 分析表
ANALYZE table_name;

-- 查看执行计划
EXPLAIN SELECT * FROM table_name WHERE condition;
```

## 12. 注意事项

1. 始终使用参数化查询来防止 SQL 注入
2. 为频繁查询的列创建索引
3. 使用事务来确保数据一致性
4. 定期备份数据库
5. 合理使用约束来保证数据完整性
6. 使用视图来简化复杂查询
7. 注意权限管理，遵循最小权限原则

# PostgreSQL 安装

## Linux上安装

1. 更新包列表
   ```bash
   sudo apt update
   ```
2. 安装 PostgreSQL
   ```bash
    sudo apt install postgresql postgresql-contrib
    ```
3. 启动 PostgreSQL 服务
    ```bash
    sudo systemctl start postgresql
    ```
4. 连接到 PostgreSQL
    ```bash
    sudo -i -u postgres     # 切换到 postgres 用户（默认超级用户）
    psql
    ```
5. 创建新用户和数据库
    ```sql
    CREATE USER myuser WITH PASSWORD 'mypassword';
    CREATE DATABASE mydb OWNER myuser;
    ```
6. 退出 psql
    ```sql
    \q
    ```
7. 退出 postgres 用户
    ```bash
    exit
    ```