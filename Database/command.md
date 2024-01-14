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

### Limit
Limit 可以对查询到的结果进一步筛选，startPostion是开始位置，length是从开始位置读取多少条记录，通常用来分页查询。

    Select *
    From table
    Where condition
    Limit startPosition, length


## Global Variable

检查端口号
```
show global variables like 'port';
```