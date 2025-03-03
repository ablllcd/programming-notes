# MySQL基础

## MySQL是什么

MySQL 是一种关系型数据库，主要用于持久化存储我们的系统中的一些数据比如用户信息。其免费开源，由Oracle提供。

其流行的原因为：

* 免费开源
* 成熟稳定
* 社区活跃
* 简单好用

### 关系型数据库
顾名思义，关系型数据库（RDB，Relational Database）就是一种建立在关系模型的基础上的数据库。关系模型表明了数据库中所存储的数据之间的联系（一对一、一对多、多对多）。

关系型数据库中，我们的数据都被存放在了各种表中（比如用户表），表中的每一行就存放着一条数据（比如一个用户的信息）。

其特点为表结构固定，定好的表结构难以更改。

### SQL 结构化查询语言

SQL 是一种结构化查询语言(Structured Query Language)，专门用来与数据库打交道，目的是提供一种从数据库中读写数据的简单有效的方法。

SQL只描述客户端希望拿到什么数据，而不描述要如何执行才能拿到数据。

## MySQL字段类型

**数值类型**：
* 整型（TINYINT、SMALLINT、MEDIUMINT、INT 和 BIGINT）
* 浮点型（FLOAT 和 DOUBLE）
* 定点型（DECIMAL）

**字符串类型**：CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT、TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB 等，最常用的是 CHAR 和 VARCHAR。

**日期时间类型**：YEAR、TIME、DATE、DATETIME 和 TIMESTAMP 等。

### Unsigned 有什么用

取消负数，从而使得正数的存储范围翻倍。

适用于从0开始不断递增的序列数。

### CHAR 和 VARCHAR 的区别是什么？

CHAR 和 VARCHAR 是最常用到的字符串类型，两者的主要区别在于：CHAR 是定长字符串，VARCHAR 是变长字符串。

* CHAR 在存储时会在右边填充空格以达到指定的长度，检索时会去掉空格；
* VARCHAR 在存储时需要使用 1 或 2 个额外字节记录字符串的长度，检索时不需要处理。
* CHAR(M) 和 VARCHAR(M) 的 M 都代表能够保存的字符数的最大值，无论是字母、数字还是中文，每个都只占用一个字符
  
CHAR 更适合存储长度较短或者长度都差不多的字符串；VARCHAR 类型适合存储长度不确定或者差异较大的字符串，例如用户昵称、文章标题等。

### VARCHAR(100)和 VARCHAR(10)的区别是什么？

VARCHAR(100)和 VARCHAR(10)都是变长类型，表示能存储最多 100 个字符和 10 个字符。当 VARCHAR(10)存储超过 10 个字符时，就需要修改表结构才可以。

* 虽说 VARCHAR(100)和 VARCHAR(10)能存储的字符范围不同，但二者存储相同的字符串，所占用磁盘的存储空间其实是一样的，这也是很多人容易误解的一点。
* 不过实际分配内存时，会分配固定大小的内存块，所以有时VARCHAR(100)可能会占用更多的内存。

### Decimal和Float的区别

Decimal是定点数，也就是小数点后边的位数是固定的，其用于高精度的运算。例如：`DECIMAL(6,2)`就是总长6位，其中小数点前4位，小数点后两位。

float的长度是固定的，而decimal的长度是由精度决定的，所以float会用精度损失的问题，而decimal不会。

### 不推荐使用text和blob

TEXT 类型类似于 CHAR，但可以存储更长的字符串，即长文本数据，例如博客内容;BLOB 类型主要用于存储二进制大对象，例如图片、音视频等文件。

在日常开发中，很少使用 TEXT 类型，但偶尔会用到，而 BLOB 类型则基本不常用。如果预期长度范围可以通过 VARCHAR 来满足，建议避免使用 TEXT。

这两种类型具有一些缺点和限制，例如：
* 不能有默认值。
* 在使用临时表时无法使用内存临时表，只能在磁盘上创建临时表。
* 检索效率较低。不能直接创建索引，需要指定前缀长度。
* 可能导致表上的 DML 操作变慢。

### DataTime和TimeStamp的区别

DATETIME 类型没有时区信息，TIMESTAMP 和时区有关。TIMESTAMP 只需要使用 4 个字节的存储空间，但是 DATETIME 需要耗费 8 个字节的存储空间。但是，这样同样造成了一个问题，Timestamp 表示的时间范围更小。

### NULL 和 '' 的区别

NULL 跟 ''(空字符串)是两个完全不一样的值，区别如下：
* NULL 代表一个不确定的值,就算是两个 NULL,它俩也不一定相等。例如，SELECT NULL=NULL的结果为 false，但是在我们使用DISTINCT,GROUP BY,ORDER BY时,NULL又被认为是相等的。
* ''的长度是 0，是不占用空间的，而NULL 是需要占用空间的。
* NULL 会影响聚合函数的结果。例如，SUM、AVG、MIN、MAX 等聚合函数会忽略 NULL 值。 COUNT 的处理方式取决于参数的类型。如果参数是 *(COUNT(*))，则会统计所有的记录数，包括 NULL 值；如果参数是某个字段名(COUNT(列名))，则会忽略 NULL 值，只统计非空值的个数。
* 查询 NULL 值时，必须使用 IS NULL 或 IS NOT NULLl 来判断，而不能使用 =、!=、 <、> 之类的比较运算符。而''是可以使用这些比较运算符的。


## 以管理员方式打开命令行
1. 可以直接搜索命令行，然后右击：以命令行方式打开
2. win+R后按ctrl+shift+enter来打开
3. 普通命令行内输入：
    `runas /user:Administrator cmd`
    后根据提示输入密码。（但是密码我忘了）

## 启动mySQL服务
1. 可以在win+R后输入services.msc查看已启动的服务
2. 通过下列命令可以启动和禁止
   ````
   // mysql80 是我电脑上mysql服务的名字，可通过services.msc查看
    net start mysql80
    net stop mysql80
   ````
## MySQL基本操作

*注意： MySQL不区分大小写* 

### 数据库外操作

#### 登录mySQL
1. 安全方式登录：
   ````
   mysql -uUSERNAME -p
   ````
    之后根据提示输入密码

2. 密码是明文
   ````
   mysql -uUSERNAME -pPASSWORD
   ````
   注意： 大写的`USERNAME`和`PASSWARD`需要替换成真实的密码。（我的root账号密码是123456）

3. 指定IP地址和端口号
   ````
   mysql -u用户名 -p密码 -hIP地址 -P端口号
   ````
   IP地址默认`localhost`，端口号默认`3306`

#### 退出mySQL
输入`exit`即可。

#### 导入SQL文件

```
mysql> create database abc;      # 创建数据库
mysql> use abc;                  # 使用已创建的数据库 
mysql> source /home/abc/abc.sql  # 导入备份数据库
```

### 全局操作

#### 数据库相关
```
// 查看数据库
show databases;
show databases like 'DBNAME';

// 创建数据库
create database DBNAME;

// 删除数据库
drop database DBNAME;

// 查看当前正在使用的数据库
select database();

// 选择使用的数据库
use DBNAME;
```

#### 全局变量相关

```
// 查看端口号
show global variables like 'port';
```

#### 隔离级别相关
```
// 全局隔离级别
select @@global.transaction_isolation;

// 当前会话隔离级别
select @@transaction_isolation;

// 修改会话隔离级别
 set session transaction isolation level serializable;
```


### DDL 语句

DDL（Data Definition Language）语句： 数据定义语言，主要是进行定义/改变表的结构、数据类型、表之间的链接等操作。常用的语句关键字有 CREATE、DROP、ALTER 等。

```
// 创建表
CREATE TABLE 表名(
列名1 数据类型,
列名2 数据类型,
列名3 数据类型,
...
)
也可以添加约束

CREATE TABLE IF NOT EXISTS `runoob_tbl`(
`runoob_id` INT UNSIGNED AUTO_INCREMENT,
`runoob_title` VARCHAR(100) NOT NULL,
`runoob_author` VARCHAR(40) NOT NULL,
`submission_date` DATE,
PRIMARY KEY ( `runoob_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

// 修改表
ALTER TABLE 表名;
eg：ALTER TABLE 表名 ADD 列名 数据类型;（添加一个列）
    ALTER TABLE 表名 CHANGE 列名 新列名 新数据类型;（修改列名）
    ALTER TABLE 表名 DROP 列名;

// 删除表
DROP TABLE 表名;
DROP DATABASE 数据库名;

```

### DML 语句
DML（Data Manipulation Language）语句: 数据操纵语言，主要是对数据进行增加、删除、修改操作。常用的语句关键字有 INSERT、UPDATE、DELETE 等。

```
// 插入
INSERT INTO table_name
VALUES (value1,value2,value3,...);

INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);

// 删除
DELETE FROM table_name WHERE condition;

// 修改
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

### DQL 语句
DQL（Data Query Language）语句：数据查询语言，主要是对数据进行查询操作。常用关键字有 SELECT、FROM、WHERE 等。

```
// 查看表信息
show tables;
describe [tableName] //查看表详细信息

// 查看表记录
Select A.*, B.name FROM A,B WHERE A.id=B.id; 
```

### DCL 语句
DCL（Data Control Language）语句： 数据控制语言，主要是用来设置/更改数据库用户权限。常用关键字有 GRANT、REVOKE 等。一般人员很少用到DCL语句。

```
GRANT （授权）

REVOKE （取消权限）
```

### 常见关键字

**primary key**：用来区分记录的key
```
ALTER TABLE Persons
ADD PRIMARY KEY (P_Id)
```

**auto increment**： 这可以更改自增长的初始值，但是好像无法将已有的列声明为自增长。
```
ALTER TABLE Persons AUTO_INCREMENT=1
```

**Limit**: 可以对查询到的结果进一步筛选，startPostion是开始位置，length是从开始位置读取多少条记录，通常用来分页查询。

```
Select *
From table
Where condition
Limit startPosition, length
```

### Join

Join可以分为多种类型：
* Join：没指定类型就等同于Cross Join（效果为卡笛尔积）
* Natural Join/inner join: 根据两表相同的列作为匹配条件来进行Join
* Left Join: Natural Join的结果+左表的其它列，缺失值写NULL
* Right Join：Natural Join的结果+右表的其它列，缺失值写NULL
* Full Join: Natural Join的结果+右表的其它列+左表的其它列，缺失值写NULL

*注意：虽然有inner和full join关键字，但其实没效果的*

Join配合的关键字：
* on：普遍的用法，其参数可以位一个字段，多个字段，甚至一个条件表达式；查询结果中common column出现两次

    ```
    select * from employee join dept_emp on employee.uid=dept_emp.uid
    ```
* using：只能指明需要连接的列；查寻结果中common column只出现一次

    ```
    select * from employee left join dept_emp using (uid)
    ```

### Aggregate

聚合通常涉及三个操作：group by；avg；having

having和where的区别在于：having是在分组后起效，where是分组前

```
select branch-name, avg (balance)
from account
group by branch-name
having avg (balance) > 1200
```

### Foreign Key 
外键就是设置表A的某列为外键，指向另一个表B的主键，从而表A和表B直接形成关系和约束。

需要注意，外键会带来性能损耗，所以不推荐使用。

**创建外键**：
```
-- 创建表时添加外键
create table 表名(
字段名 数据类型,
...
[constraint] [外键名称] foreign key (外键字段名) references 主表 (主表列名)
);
 
-- 单独添加外键
alter table 表名 add constraint 外键名称 foreign key (外键字段名)
references 主表 (主表列名) ;
 
-- 删除外键
alter table 表名 drop foreign key 外键名称;
```

**外键约束**

外键约束可以保证数据的完整性和一致性：

假设有两个表： students （学生表）和 courses （课程表）。每个学生可以选择多门课程，因此我们希望通过外键约束来确保学生表中的 course_id 列与课程表中的 course_id 列保持一致。

1. 数据完整性：通过外键约束，我们可以确保学生表中的 course_id 列只引用了课程表中存在的有效 course_id 值。这样可以防止无效的或不存在的课程ID被插入到学生表中，保证数据的完整性。

2. 数据一致性：外键约束可以确保学生表中的 course_id 列与课程表中的 course_id 列保持一致。如果在课程表中更新或删除了某门课程的记录，外键约束会自动处理相关的学生表中的数据，以保持数据的一致性。

**外键删除**

当主表中的数据删除时，mysql有如下选项来处理删除行为：

* no action： 当在父表（主表）中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。 (与 RESTRICT 一致) 默认行为
* restrict： 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。 (与 NO ACTION 一致) 默认行为
* cascade：当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有，则也删除/更新外键在子表中的记录。
* set null： 当在父表中删除对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为null（这就要求该外键允许取null）。
* set default：父表有变更时，子表将外键列设置成一个默认的值 (Innodb不支持)

```
alter table 子表名 add constraint 外键名称 foreign key (子表字段名) references 父表名(父表字段名) on update 行为 on delete 行为;
```


## 常见问题

### 数据库中的中文记录在命令行中显示乱码

解决方法：登录时设置默认字符集即可

```
mysql -u root -p --default-character-set=utf8
```

# MySQL高级篇

## 整体架构

![alt text](mysqlPIC/整体框架.png)

MySQL服务器如图分为四层：
* 连接层：处理客户端的链接，用户的登录校验等
* 服务层：处理真正的服务请求，例如对表的增删改查
* 引擎层：控制数据存储和查询的方式
* 存储层：负责数据的物理存储

执行流程为：

![alt text](mysqlPIC/sql执行流程.png)

* 连接器： 身份认证和权限相关(登录 MySQL 的时候)。
* 查询缓存： 执行查询语句的时候，会先查询缓存（MySQL 8.0 版本后移除，因为这个功能不太实用）。
* 分析器： 没有命中缓存的话，SQL 语句就会经过分析器，分析器说白了就是要先看你的 SQL 语句要干嘛，再检查你的 SQL 语句语法是否正确。
* 优化器： 按照 MySQL 认为最优的方案去执行。
* 执行器： 执行语句，然后从存储引擎返回数据。 执行语句之前会先判断是否有权限，如果没有权限的话，就会报错。


## 存储引擎

存储引擎负责数据存储，索引创建，以及查询/更新指令的实际执行。由于存储引擎是作用于`表`的，而不是整个数据库的，所以也被成为`表类型`。

默认的存储引擎为InnoDB，也可以在创建表时指定，语句为：
```
 CREATE TABLE `runoob_tbl` (
  `runoob_id` int unsigned NOT NULL AUTO_INCREMENT,
  `runoob_title` varchar(100) NOT NULL,
  `runoob_author` varchar(40) NOT NULL,
  `submission_date` date DEFAULT NULL,
  PRIMARY KEY (`runoob_id`)
) ENGINE=InnoDB;
```

也可以查看当前MySQL软件支持的存储引擎，语句为：
```
show engines;
```


### Innodb 存储引擎
InnoDB 是 MySQL 中默认的存储引擎,它拥有以下一些重要的特点:

1. 事务支持：InnoDB 提供了完整的事务支持,支持 ACID 特性,可以确保数据的完整性和一致性。
2. 行级锁：InnoDB 使用行级锁定,而不是表级锁定,这大大提高了多用户并发操作的性能。
3. 外键约束：InnoDB 支持外键约束,可以方便地定义表与表之间的关系。
4. 崩溃恢复：数据库崩溃后，可以恢复到崩溃前。（通过redo log）

innoDB引擎的每张表都会对应这样一个表空间文件`xxx.ibd`：
* xxx代表的是表名
* 该文件存储xxx表的表结构(frm、sdi)、数据和索引
* 可以通过`show variables like 'innodb_file_per_table'`来查看innodb_file_per_table，如果是ON则表示每张表都有单独的idb文件
* 本机的idb文件存储在C:\ProgramData\MySQL\MySQL Server 8.0\Data

innoDB的逻辑存储结构如下：

![alt text](mysqlPIC/2.png)

* 其中区固定为1M，页固定为16K，所以一个区内有64页
* Row中存储的数据除了column外，还有额外的数据

这里先简单了解下，后续优化时会用到。

### MyISAM 和 Memory 存储引擎

MyISAM是早期的默认引擎，其特点为：
1. 不支持事务
2. 表级锁
3. 不支持外键
4. 当数据库崩毁后，不支持安全恢复

MyISAM类型的表强调的是性能，但是不提供事务支持，所以当cup核心数增高时，性能反而不如InnoDB。

MyISAM对应的表在磁盘上存储成三个文件：
* .frm文件存储表定义。
* 数据文件的扩展名为.MYD (MYData)。
* 索引文件的扩展名是.MYI (MYIndex)。

Memory将数据存储在内存中，作为临时存储表，可以使用hash索引

### 存储引擎选择

Innodb: 在需要事务或者需要保证数据一致性时使用。

MyISAM: 可用于对数据一致性要求不高的地方，例如频繁的插入和删除操作，少量的更改操作。（

Memory：可用于缓存。

而由于NoSQL的存在，让MongoDB替代了MyISAM，Redis替代了Meomory。

## 索引

复习索引可看：https://javaguide.cn/database/mysql/mysql-index.html#%E7%B4%A2%E5%BC%95%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B9

MySQL中索引是由存储引擎负责的，而关于索引的基本知识这里就不重复了，常见的索引有B+树，红黑树，hash表。

默认选取B+树作为索引的原因：

1. 红黑树是二叉树，树的深度高，对应的IO次数也多
2. hash表不适合动态结构，需要时常更新hash函数；此外hash索引是无序的，无法进行范围查找或者排序

索引也有一些使用原则：

1. 对于经常进行查询的字段，为其构建索引
2. 对于group by, order by的字段，为其构建索引
3. 相较于单个索引，联合索引的性能更好，因为可以避免回表查询（二次查询索引）


### FullText 索引

对于文本类型的字段，可以构建全文索引，从而方便搜索等功能，其原理如下：

1. 分词处理：在建立 FullText 索引之前,MySQL 会先对文本内容进行分词处理。
分词器会将文本拆分成一个个独立的词语单元。
常见的分词器有 InnoDB 内置的 ngram 分词器和 MyISAM 的 FULLTEXT 分词器。
2. 建立倒排索引：分词之后,MySQL 会为每个词语建立一个倒排索引。
倒排索引会记录每个词语出现在哪些文档(行记录)中,以及出现的位置信息。
这样可以快速定位包含某个搜索词的行记录。
3. 存储索引数据： 构建好倒排索引后,MySQL 会将这些索引数据存储在磁盘上。
对于 InnoDB 表,索引数据会存储在表空间中。
对于 MyISAM 表,索引数据会单独存储在索引文件中。
4. 查询处理： 用户进行全文搜索查询时,MySQL 会先对查询语句进行分词。
然后根据分词结果,查找倒排索引中相关的文档(行记录)。
最后汇总各个搜索词的结果,返回最终的查询结果。
5. 评分排序： FullText 索引支持对查询结果进行相关性评分和排序。
评分规则可以根据各个搜索词在文档中的出现频率、位置等因素来确定。
这样可以将最相关的结果排在前面,提高搜索的准确性。

（内容来自GPT，不保证准确性）

### 前缀索引

为varchar, text等文本数据创建索引时，默认还是用b+树构建索引，key是整个文本。而由于文本内容通常较大，会导致整个索引表很大，所以提示前缀索引来解决key太大的问题。

前缀索引的思路很简单，还是使用b+树构建索引，只是key不是整个文本，而是文本的前缀。语法：

```
create index [index_name] on [table_name]([column_name](n))
```
其中column_name后边跟的n就是前缀的长度。前缀越长，索引的区分度越高；前缀越短，索引表越小。

### 联合索引

有时候我们可以拿多个字段来创建索引，这就是联合索引:

![alt text](mysqlPIC/5.png)


### 聚集（一级）索引和二级索引

![alt text](mysqlPIC/3.png)

一级索引的特点：
1. 实际数据按照一级索引的key进行排序，所有叶子节点的顺序与实际数据的顺序一致，方便进行范围查找
2. 为了保持顺序性，如果在中间插入数据3，需要挪动大于3的实际数据的位置
3. MySQL中有且只有一个聚集索引，通常选primary key来构建

二级索引的特点：
1. 索引的叶子节点和实际数据的顺序并不对应
2. 叶子节点存储的是一级索引的key（没有存储实际数据的指针是因为实际数据的位置可能变动）
3. 数据库中可以有多个二级索引
4. 每次查询二级索引得到一级索引的id后，还需要再去一级索引进行一次查询

### 索引指令

1. 查看索引
    ```
    show index from [table_name];
    ```

2. 创建索引
   ```
   create index [index_name] on [table_name]([column1],[column2]...)
   ```

3. 删除索引
   ```
   DROP INDEX index_name ON table_name;
   ```

### 性能分析

**查看sql执行次数**

```
show global status like 'Com_______';
```

通过查看该数据库全局信息来判断哪种类型的查询占主导地位，从而进行性能优化。

**查看慢查询日志**

```
show variables like 'slow_query_log';
```

通过上述语句可以查看`慢查询日志`是否开启，可以通过配置文件修改`慢查询`的阈值，对于超过该阈值的查询指令会被记录到日志中。（配置文件怎么改自行百度，不同环境的操作应该不同）


**profiling分析**
```
select @@have_profiling;    //是否支持profiling
select @@profiling;         //是否开启了profiling
set profiling=1;            //开启profiling
show profiles;              //显示profiles
show profile for query [query_id];      //查看某条语句的详细执行情况
```

profile会显示所有sql语句的执行耗时。

**explain分析**

上述profiles只是显示sql的耗时，而耗时高未必代表该sql语句差。为此，还有explain(describe)来分析sql语句的执行流程：

```
explain [sql语句]
```

![alt text](mysqlPIC/4.png)

1. id: 表示表查询的顺序，id越大越先执行，id相同按先后顺序执行；（单表查询只会有一个id）
2. select type: 表示该表查询的类别，例如simple-简单表查询；primary-主查寻，外层查询；subquery-子查询；
3. type: 连接类型，也是主要优化目标。
4. possible_key: 该表存在的索引
5. key: 该表用到的索引
6. rows: 预计要查询的行数
7. filtered: 实际用到的行数 / 查询的行数；值越大越好


### 索引失效

在一千万条数据中，如果没有对age字段创建索引，那么根据age查找就是全局扫描，耗时约20s；而再对age创建索引后，根据age查找只需要0.01s。然而在一些情况下，有索引也会走全文搜索：


**联合索引的失效问题**

在创建索引时，可以为att1,att2,att3三个字段创建一个联合索引。如果查询条件包含att1,att2,att3自然没有问题，直接根据索引进行查询，而如果查询条件中缺少了某个字段，则需要满足`最左前缀法则`。

例如我们创建联合索引时的顺序为att1,att2,att3，那么查询条件中提供的字段也需要从左向右匹配。以下是一些示例：

1. ```select * from table where att1='a', att2='a';```  此时提供了att1和att2，满足att1,att2,att3的顺序，那么att1和att2都根据索引查找

2. ```select * from table where att1='a', att3='a';``` 此时att1能满足，而att2时不满足了，那么只有att1能走索引，att3进行全文搜索。

3. ```select * from table where att2='a', att3='a';``` 此时从att1就不满足了，那么att2和att3都走全文搜索。

此外，某个字段进行范围查询时，也会导致后续索引失效，例如：

```
select * from table where att1='a', att2>10, att3='a';
```

此时att3的查找不走索引，而将>更改为>=即可解决问题

```
select * from table where att1='a', att2 >= 10, att3='a';
```

此时三个字段都走索引。


**运算操作**

如果给字段att1添加索引，那么根据att1查找会走索引。然而，如果根据operate(att1)进行查找则不走索引，例如：

```
select * from table where substring(att1,10,2) = '15';
```

原因是：如果等号两边的数据类型不一致，则会发生隐式转换。

例如，explain select * from evt_sms where phone = 13020733815;这条SQL语句就会变为explain select * from evt_sms where cast(phone as signed int) = 13020733815;

由于对索引列进行了函数操作，从而导致索引失效。


**字符串没加引号**

对于字符串类型，加不加单引号都能返回正确结果，然而不加单引号是不走索引的， 例如:

```
select * from table where phone='123456';   //走索引
select * from table where phone=123456;   //不走索引
```

**模糊匹配**

如果只是尾部模糊则走索引，如果包含了头部模糊则全局搜索：

```
select * from table where phone like '12%';   //走索引
select * from table where phone like '%56';   //不走索引
select * from table where phone like '%34%';   //不走索引
```

**or和and连接**

如果attr1有索引,att2没有索引，那么根据 att1='a' or att2='a'搜索则都不走索引。

如果att1和att2都有索引，两者用and连接，此时也只会选择使用一个索引进行查找，另一个字段进行全文匹配。


**数据分布**

如果mysql判断走索引没有走全局扫描快，则不走索引，例如某个字段的条件只能过滤掉10%的数据，那么mysql不走该字段的索引，而是直接进行全局扫描。

### MySQL提示

有时一个字段可能既有单独的索引，又参与了联合索引，那么此时mysql需要自己决定使用哪个索引，而MySQL提示就是帮助MySQL选择索引。

语法：
```
select [condition] from [table_name] use [index_name] where [condition]
select [condition] from [table_name] ignore [index_name] where [condition]
select [condition] from [table_name] force [index_name] where [condition]
```

这里有三个关键词：use表示建议使用该索引，ignore表示不用该索引，force表示强制使用该索引。


### 缩小select范围

通常我们习惯使用select *，但实际上如果我们需要的数据量比较小，尽量就缩小select的范围。

例如： where condition中我们根据att1字段进行索引得到了att1的值和id值，如果此时我们只需要att1和id，那么只进行一次索引查询即可；如果我们使用select *，那么还需要根据第一次索引得到的id进行回表查询，增加开销。

此外，如果我们需要查询att1和att2两个属性，而where condition中只使用att1，此时我们可以构建联合索引（att1,att2），这样在索引中就可以得到att1和att2.


## 优化

### insert 优化

关于插入优化有这几点：

1. 批量插入：避免与数据库多次建立连接的开销
    ```
    INSERT INTO TABLE VALUES ([value1]),([value2])
    ```

2. 手动开启事务：mysql默认自动提交，在多次插入时手动开启事务可以减少事务开启和关闭的开销
    ```
    start transaction
    [insert sql1]
    [insert sql2]
    [insert sql3]
    commit;
    ```

3. 主键顺序插入

4. 大量数据插入使用load效率更高
   ```
   LOAD DATA LOCAL INFILE [local file path] INTO TABLE [table_name]
   FIELDS TERMINATED BY ','
   LINES TERMINATED BY '\n';
   ```

### 主键优化

上述插入时我们提及到主键插入最好是顺序插入，这是因为聚合索引的原因。其中数据会按照主键进行顺序存储，如果乱序插入的话，会导致已存储的数据反复移动。此外，由于数据是存储在`页`上的，当数据进行删除时，页中存储的数据变少，可能会进行与邻接的页进行合并。其中可以设置MERGE_THRESHOLD来设置页数据省多少时进行合并。

主键设计原则：

1. 主键长度尽量小：因为其他二级索引的叶子节点存储的都是主键
2. 顺序插入，尽量使用auto_increment关键字保证顺序
3. 避免修改主键，因为要修改所有的索引

### order by 优化

MySQL排序有两种类型：filesort和using index。其中filesort是通过索引或者全局扫描获取数据后，放入到缓冲区然后进行排序；而using index是通过读取索引可以直接获取到排序好的数据。所以我们可以设计索引和查询语句来保证using index类型的排序。

其中排序优化原则有：

1. 能用覆盖索引尽量用覆盖索引。覆盖索引是指查询的字段在索引表中都能拿到，无需回表查询去读取实际的行数据。只有覆盖索引才有可能是using index类型的排序。

2. 查询多个字段时或者多字段排序时使用联合索引，根据排序时要求字段的升序和降序来设计联合索引中字段的升序和降序（默认都为升序）

3. 当不可避免filesort时，如何需要排序的数据量大，可以拓展缓冲区的大小，从而避免磁盘读写。

### group by 优化

思路和order by一致，也是通过覆盖索引来避免查询实际数据，直接通过索引完成分组查询。其中分组也需要注意联合索引的最左匹配问题。

### limit 优化

```
select * from table limit 10000, 10
```

当我们要从10000的位置向后读取10条数据时，我们发现此时的开销已经较大，而且随着limit后边的10000继续增大时，开销也会继续增大。

这是因为mysql要读取前10010条实际数据，进行排序然后返回10条数据。其中读取10010条实际数据的开销是很大的。对应的优化方案则是：索引查询+子查询（联合查询）

```
select * from table where id in (select id from table limit 10000,10)
```

其中先获取limit数据对应的id，该步骤通过索引即可完成；然后再根据id获取对应的实际数据，从而避免了读取大量的实际数据。

## 视图

视图(View)是一种虚拟存在的表。视图中的数据并不在数据库中实际存在,行和列数据来自定义视图的查询中使用的表,并且是在使用视图时动态生成的。

通俗的讲,视图只保存了查询的SQL逻辑,不保存查询结果。所以我们在创建视图的时候,主要的工作就落在创建这条SQL查询语句上。

### 基本使用

```
## 创建视图
create view user_v1 as select id,username from user;

## 查询视图
select * from user_v1 where id=1;
select * from user_v1;

## 修改视图
update user_v1 set username='ccain' where id=1;
insert into user_v1 value (3,'xiaoming');

## 删除视图
drop view if exists user_v1;
```

需要注意：对视图的修改会同步到真正的user表上，对user表的修改视图也会同步更新。

### 检查选项

```
## 检查选项
create view user_v1 as select id,username from user where id<20 with cascaded check option;
insert into user_v1 value (30,'xiaoming');
```

此时的插入会失败，因为id30不满足创建view时where的限制条件。当使用WITH CHECK OPTION子句创建视图时,MySQL会通过视图检查正在更改的每个行,例如插入,更新,删除,以使其符合视图的定义。

此外，MySQL允许基于另一个视图创建视图,新建视图还会检查依赖视图中的规则以保持一致性。为了确定检查的范围,mysql提供了两个选项:CASCADED 和LOCAL,默认值为 CASCADED。

#### CASCADE

```
create or replace view user_v1 as select id,username from user where id<20;
create view user_v2 as select id,username from user_v1 where id>10 with cascaded check option;

## 失败：因为不满足v1要求
insert into user_v2 value (30,'xiaoming');
## 成功：因为下级不用考虑上级的约束
insert into user_v1 value (5,'xiaoming');
```

CASCADE表示级联检查，对当前视图更新时，需要保证满足所有下级表的要求（哪怕下级表没有加check option）。如果当前表没加with check option，但是有下级表，也是需要满足下级表的要求。

![alt text](../Java/pic/CASCADE.png)

#### LOCAL

![alt text](../Java/pic/LOCAL-CHECK.png)

Local也需要进行递归检查，对当前视图更新时，需要检查下级表的要求：如果下级表没有加check option，则无需满足下级表的要求；如果有则必须满足其要求。如果当前表没加with check option，但是有下级表，也是需要按照上述规则对下级表的要求进行检查。

### 视图更新要求

要使视图可更新,视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含以下任何一项,则该视图不可更新:

1. 聚合函数或窗口函数(SUM()、MIN()、MAX()、COUNT()等)
2. DISTINCT
3. GROUP BY
4. HAVING
5. UNION 或者 UNION ALL

### 视图的作用


1. 简化操作：通过视图将一些数据抽象出来后，可以直接对视图进行操作

2. 安全：数据库可以授权,但不能授权到数据库特定行和特定的列上，只能进行到表上。通过创建视图，然后授权用户可以看到哪个视图，从而进行更细致的授权。（例如创建视图只显示学生的基本信息，而不显示账号密码）

3. 数据独立：视图可帮助屏蔽真实表结构变化带来的影响。（例如真实表的列名修改，我们只需要修改创建视图的语句即可，前端看到的视图不变化）

## 存储过程

存储过程是事先经过编译并存储在数据库中的一段SQL语句的集合,调用存储过程就会执行这些SQL语句。从而简化应用开发人员的工作,并减少数据在数据库和应用服务器之间的网络传输。（不需要多次通过网络执行SQL语句）

存储过程思想上很简单,就是数据库SQL语言层面的代码封装与重用。

### 基本使用

```
## 创建
create procedure p1()
begin
    select count(*) from user;
end;

## 调用
call p1();

## 删除
drop procedure p1;
```

注意：命令行中创建procedure时，其中使用了多个';'，导致命令行在遇到第一个';'时就认为输入结束了。为此需要使用delimiter来修改结束符。

### 变量

#### 系统变量

系统变量是MySQL服务器提供,不是用户定义的,属于服务器层面。分为全局变量(GLOBAL)、会话变量(SESSION)。

```
## 查看系统变量，默认为session级别
show global variables ;
show session variables ;

## 模糊搜索
show variables like 'auto%';

## 查看具体变量
select @@autocommit;

## 修改系统变量
set @@session.autocommit=0;
```

#### 用户变量

用户定义变量是用户根据需要自己定义的变量。用户变量不用提前声明,在用的时候直接用“@变量名”使用就可以，其作用域为当前连接。

```
## 直接赋值
set @myName = 'cain';
set @myAge := 22;

## 查询后赋值，每个变量只能存储一个值，无法存储数组
select count(*) into @myCount from user;

## 使用用户变量
select @myName,@myCount,@myUsers;
```

#### 局部变量

局部变量 是根据需要定义的在局部生效的变量,访问之前,需要DECLARE声明。局部变量的范围是在其内声明的BEGIN ... END块。

```
create procedure p1()
begin
    ## 声明
    declare user_count int default 0;
    ## 赋值
    select count(*) into user_count from user;
    ## 使用
    select user_count;
end;

call p1();
```

### 循环&判断

有点无聊，具体用到在查吧。

### 输入输出参数

* IN: 该类参数作为输入,也就是需要调用时传入值
* OUT: 该类参数作为输出,也就是该参数可以作为返回值
* INOUT: 既可以作为输入参数,也可以作为输出参数

```
create procedure p1(in score int, out result varchar(10))
begin
    if score<60 then
        set result:='不及格';
    else
        set result:='及格';
    end if;
end;

call p1(70,@my_result);
select @my_result;
```
### 存储函数

存储函数是有返回值的存储过程，且存储函数的参数只能是IN类型的。（其实就是另一种格式的存储过程，没啥区别）

具体语法如下:
```
CREATE FUNCTION 存储函数名称([参数列表])
RETURNS typb [characteristic ... ]
BEGIN

-- SQL语句
RETURN ...;

END ;

characteristic说明:
· DETERMINISTIC:相同的输入参数总是产生相同的结果
· NO SQL:不包含SQL语句。
· READS SQL DATA:包含读取数据的语句,但不包含写入数据的语句。
```