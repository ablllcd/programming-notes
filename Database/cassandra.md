## 什么是 Cassandra

Apache Cassandra 是一个开源的分布式 NoSQL 数据库系统，最初由 Facebook 开发。它以高可用性、可扩展性和容错性著称，适用于处理大规模结构化数据。Cassandra 采用无中心化的对等架构，支持多数据中心复制，能够实现无单点故障的数据存储。其数据模型基于宽列存储，适合写密集型和高吞吐量的应用场景，如物联网、日志收集和实时分析等。

**主要特点：**
- 分布式与去中心化架构
- 高可用性与容错性
- 可横向扩展
- 支持多数据中心复制
- 灵活的 Schema 设计

## 核心概念

- **节点（Node）**：Cassandra 集群中的单个服务器。
- **集群（Cluster）**：由多个节点组成的集合，协同存储和管理数据。
- **数据中心（Data Center）**：节点的逻辑分组，通常对应物理位置或用途。
- **键空间（Keyspace）**：类似于关系型数据库的数据库，定义数据的复制策略。
- **表（Table）**：存储数据的结构，类似于关系型数据库的表。
- **分区键（Partition Key）**：决定数据分布到哪个节点的键。
- **列族（Column Family）**：Cassandra 的表结构，支持宽列存储。
- **一致性级别（Consistency Level）**：控制读写操作的数据一致性要求。

在 Cassandra 中，“节点（Node）”通常指的是集群中的一台服务器（物理机或虚拟机）。每个节点都运行 Cassandra 实例，负责存储和管理部分数据。你可以把“一台电脑 = 一个节点”理解为最常见的部署方式。每个节点之间是对等的，没有主从之分。多个节点组成集群，共同实现数据的分布式存储和高可用性

## 基本操作

Cassandra 使用 CQL（Cassandra Query Language）进行数据操作，语法类似 SQL。

**创建键空间：**
```sql
CREATE KEYSPACE my_keyspace WITH REPLICATION = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};
```

**创建表：**
```sql
CREATE TABLE my_keyspace.users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    age INT
);
```

**插入数据：**
```sql
INSERT INTO my_keyspace.users (user_id, name, age)
VALUES (uuid(), 'Alice', 30);
```

**查询数据：**
```sql
SELECT * FROM my_keyspace.users WHERE user_id = ...;
```

**更新数据：**
```sql
UPDATE my_keyspace.users SET age = 31 WHERE user_id = ...;
```

**删除数据：**
```sql
DELETE FROM my_keyspace.users WHERE user_id = ...;
```

**查看表结构：**
```sql
DESCRIBE TABLE my_keyspace.users;
```
该命令会显示表的列、主键、索引等详细结构信息。

## 列的更改

Cassandra 支持对表的列进行更改。你可以使用 `ALTER TABLE` 语句添加新列或删除已有列，但不能直接修改已有列的数据类型（需要先删除再添加）。常见操作如下：

**添加列：**
```sql
ALTER TABLE my_keyspace.users ADD email TEXT;
```

**删除列：**
```sql
ALTER TABLE my_keyspace.users DROP age;
```

注意：删除列后，已有数据中的该列内容会被标记为删除（Tombstone），实际物理删除会在后台进行。