# MySQL related Command

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

## 登录mySQL
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

## 退出mySQL
输入`exit`即可。

## Database level command
（懒得切换中英文，所有下边一般就用英文了，全大写的名称一般都是要替换为对应的变量名） 

*Note： there is no difference between UPPER CASE and lower case in mySQL* 
### Show Database
    //show all database
    show databases;
    //show specific database
    show databases like 'DBNAME';

### Create Database
    create database DBNAME;

### Delete Database
    drop database DBNAME;

### Check current databse
    select database();

### Change current databse
    use DBNAME;



## Table Level
### Create Table

    CREATE TABLE table_name
    (
    column_name1 data_type(size),
    column_name2 data_type(size),
    column_name3 data_type(size),
    ....
    )

也可以添加约束

    CREATE TABLE IF NOT EXISTS `runoob_tbl`(
    `runoob_id` INT UNSIGNED AUTO_INCREMENT,
    `runoob_title` VARCHAR(100) NOT NULL,
    `runoob_author` VARCHAR(40) NOT NULL,
    `submission_date` DATE,
    PRIMARY KEY ( `runoob_id` )
    )ENGINE=InnoDB DEFAULT CHARSET=utf8;


### Show tables of the database
    show tables;

    describe [tableName] //查看表详细信息

### Constrain
primary key

    ALTER TABLE Persons
    ADD PRIMARY KEY (P_Id)

auto increment： 这可以更改自增长的初始值，但是好像无法将已有的列声明为自增长。

    ALTER TABLE Persons AUTO_INCREMENT=1

## Record Level
### Insert

    INSERT INTO table_name
    VALUES (value1,value2,value3,...);

    INSERT INTO table_name (column1,column2,column3,...)
    VALUES (value1,value2,value3,...);

### Delete

    DELETE FROM table_name
    WHERE condition;

### Modify

    UPDATE table_name
    SET column1 = value1, column2 = value2, ...
    WHERE condition;

### Select
    // 有的时候我们可能希望使用*和特定列来完成查询
    Select A.*, B.name FROM A,B WHERE A.id=B.id; 

### Limit
Limit 可以对查询到的结果进一步筛选，startPostion是开始位置，length是从开始位置读取多少条记录，通常用来分页查询。

    Select *
    From table
    Where condition
    Limit startPosition, length

### Join

Join可以分为多种类型：
* Join：没指定类型就等同于Cross Join
* Cross Join：效果为卡笛尔积
* Natural Join: 就是Natural Join(效果等同于inner join)
* Left Join: Natural Join的结果+左表的其它列，缺失值写NULL
* Right Join：Natural Join的结果+右表的其它列，缺失值写NULL
* Full Join: Left Join的结果 Union Right Join的结果。（注意：mysql中虽然有full join但其效果等同于join，所以要实现真正的full join效果需要自己union。）

*注意：虽然有inner和full join关键字，但其实没效果的*

Join配合的关键字：
* on：需要指明连接条件，有重复的列（在Full join时必须用这种，因为需要重复的列！）

    ```
    select * from employee join dept_emp on employee.uid=dept_emp.uid
    ```
* using:指明需要连接的列，没有重复的列

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

## Global Variable

检查端口号
```
show global variables like 'port';
```

# MySQL高级篇

## 整体架构

![alt text](mysqlPIC/整体框架.png)


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

其逻辑存储结构如下：

![alt text](mysqlPIC/2.png)

### MyISAM 和 Memory 存储引擎

MyISAM是早期的默认引擎，其特点为：
1. 不支持事务
2. 表级锁
3. 不支持外键

Memory将数据存储在内存中，作为临时存储表，可以使用hash索引

### 存储引擎选择

Innodb: 在需要事务或者需要保证数据一致性时使用。

MyISAM: 可用于对数据一致性要求不高的地方，例如频繁的插入和删除操作，少量的更改操作。（日志系统）

Memory： 可用于缓存。

而由于NoSQL的存在，让MongoDB替代了MyISAM，Redis替代了Meomory。


## 索引

MySQL中索引是由存储引擎负责的，而关于索引的基本知识这里就不重复了，常见的索引有B+树，红黑树，hash表。

默认选取B+树作为索引的原因：

1. 红黑树是二叉树，树的深度高，对应的IO次数也多
2. hash表不适合动态结构，需要时常更新hash函数；此外hash索引是无序的，无法进行范围查找或者排序
   
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